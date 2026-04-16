<script>
    import LogoMark from "$lib/components/LogoMark.svelte";

    function initCanvas(canvasEl) {
        const ctx = canvasEl.getContext("2d");
        const header = canvasEl.parentElement;
        let animId;
        let rotation = 0;
        let mx = 0, my = 0;
        let tiltX = 0, tiltY = 0;
        const PI2 = Math.PI * 2;
        const hexChars = "0123456789ABCDEF";

        // Generate hex data blocks
        const hexBlocks = [];
        for (let i = 0; i < 8; i++) {
            let line = "";
            for (let j = 0; j < 20; j++) {
                line += hexChars[Math.floor(Math.random() * 16)];
                line += hexChars[Math.floor(Math.random() * 16)];
                if (j < 19) line += " ";
            }
            hexBlocks.push(line);
        }

        // Security labels
        const labels = [
            "THREAT LEVEL: LOW", "PERIMETER: SECURE", "ENCRYPTION: AES-256",
            "PROTOCOL: ACTIVE", "SIGNAL: ENCRYPTED", "STATUS: MONITORING",
            "CLEARANCE: ALPHA", "FREQ: 142.8 MHz"
        ];

        // Floating data points on globe
        const dataPoints = [];
        for (let i = 0; i < 30; i++) {
            dataPoints.push({
                lat: (Math.random() - 0.5) * Math.PI,
                lon: Math.random() * PI2,
                pulse: Math.random() * PI2,
                size: 1 + Math.random() * 2,
            });
        }

        // Connection lines between some points
        const connections = [];
        for (let i = 0; i < 12; i++) {
            connections.push([
                Math.floor(Math.random() * dataPoints.length),
                Math.floor(Math.random() * dataPoints.length),
            ]);
        }

        function resize() {
            const rect = header.getBoundingClientRect();
            canvasEl.width = rect.width;
            canvasEl.height = rect.height;
        }

        resize();
        window.addEventListener("resize", resize);

        header.addEventListener("mousemove", (e) => {
            const rect = header.getBoundingClientRect();
            mx = (e.clientX - rect.left) / rect.width;
            my = (e.clientY - rect.top) / rect.height;
        });

        function project(lat, lon, cx, cy, r) {
            // Apply mouse tilt
            const cosT = Math.cos(tiltX);
            const sinT = Math.sin(tiltX);
            const cosP = Math.cos(tiltY);
            const sinP = Math.sin(tiltY);

            // Spherical to cartesian
            let x0 = Math.cos(lat) * Math.sin(lon + rotation);
            let y0 = Math.sin(lat);
            let z0 = Math.cos(lat) * Math.cos(lon + rotation);

            // Rotate around X axis (vertical mouse)
            let y1 = y0 * cosT - z0 * sinT;
            let z1 = y0 * sinT + z0 * cosT;

            // Rotate around Y axis (horizontal mouse)
            let x1 = x0 * cosP + z1 * sinP;
            let z2 = -x0 * sinP + z1 * cosP;

            return { x: cx + r * x1, y: cy - r * y1, z: z2 };
        }

        function drawGlobe(cx, cy, r, alpha) {
            // Outer ring
            ctx.strokeStyle = `hsla(200, 60%, 45%, ${alpha * 0.4})`;
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.arc(cx, cy, r, 0, PI2);
            ctx.stroke();

            // Second ring
            ctx.strokeStyle = `hsla(200, 60%, 45%, ${alpha * 0.15})`;
            ctx.beginPath();
            ctx.arc(cx, cy, r * 1.08, 0, PI2);
            ctx.stroke();

            // Latitude lines
            for (let lat = -60; lat <= 60; lat += 30) {
                const latRad = (lat * Math.PI) / 180;
                ctx.strokeStyle = `hsla(200, 50%, 50%, ${alpha * 0.15})`;
                ctx.lineWidth = 0.5;
                ctx.beginPath();
                let started = false;
                for (let lon = 0; lon <= 360; lon += 5) {
                    const lonRad = (lon * Math.PI) / 180;
                    const p = project(latRad, lonRad, cx, cy, r);
                    if (p.z > 0) {
                        if (!started) { ctx.moveTo(p.x, p.y); started = true; }
                        else ctx.lineTo(p.x, p.y);
                    } else {
                        started = false;
                    }
                }
                ctx.stroke();
            }

            // Longitude lines
            for (let lon = 0; lon < 360; lon += 30) {
                const lonRad = (lon * Math.PI) / 180;
                ctx.strokeStyle = `hsla(200, 50%, 50%, ${alpha * 0.15})`;
                ctx.lineWidth = 0.5;
                ctx.beginPath();
                let started = false;
                for (let lat = -90; lat <= 90; lat += 5) {
                    const latRad = (lat * Math.PI) / 180;
                    const p = project(latRad, lonRad, cx, cy, r);
                    if (p.z > 0) {
                        if (!started) { ctx.moveTo(p.x, p.y); started = true; }
                        else ctx.lineTo(p.x, p.y);
                    } else {
                        started = false;
                    }
                }
                ctx.stroke();
            }

            // Data points
            const time = Date.now() * 0.001;
            const projected = dataPoints.map(dp => {
                const p = project(dp.lat, dp.lon, cx, cy, r);
                return { ...p, ...dp };
            });

            for (const dp of projected) {
                if (dp.z <= 0) continue;
                const pulse = 0.5 + 0.5 * Math.sin(time * 2 + dp.pulse);
                const s = dp.size * (1 + pulse * 0.5);
                const a = alpha * (0.3 + dp.z * 0.5) * (0.6 + pulse * 0.4);
                ctx.fillStyle = `hsla(200, 70%, 60%, ${a})`;
                ctx.beginPath();
                ctx.arc(dp.x, dp.y, s, 0, PI2);
                ctx.fill();

                // Pulse ring
                if (pulse > 0.7) {
                    ctx.strokeStyle = `hsla(200, 70%, 60%, ${a * 0.3})`;
                    ctx.lineWidth = 0.5;
                    ctx.beginPath();
                    ctx.arc(dp.x, dp.y, s + 4 * pulse, 0, PI2);
                    ctx.stroke();
                }
            }

            // Connection arcs between points
            ctx.lineWidth = 0.5;
            for (const [a, b] of connections) {
                const pa = projected[a];
                const pb = projected[b];
                if (pa.z <= 0 || pb.z <= 0) continue;
                const connAlpha = alpha * Math.min(pa.z, pb.z) * 0.25;
                ctx.strokeStyle = `hsla(200, 60%, 55%, ${connAlpha})`;
                ctx.beginPath();
                ctx.moveTo(pa.x, pa.y);
                // Curved arc through center-ish
                const midX = (pa.x + pb.x) / 2 + (pa.y - pb.y) * 0.15;
                const midY = (pa.y + pb.y) / 2 + (pb.x - pa.x) * 0.15;
                ctx.quadraticCurveTo(midX, midY, pb.x, pb.y);
                ctx.stroke();
            }
        }

        function drawHUD(cx, cy, r, alpha) {
            const time = Date.now() * 0.001;

            // Hex data blocks — bottom right of globe
            ctx.font = "9px monospace";
            const hx = cx + r * 0.6;
            const hy = cy + r * 0.7;
            for (let i = 0; i < hexBlocks.length; i++) {
                // Mutate some chars slowly
                if (Math.random() < 0.02) {
                    const arr = hexBlocks[i].split("");
                    const pos = Math.floor(Math.random() * arr.length);
                    if (arr[pos] !== " ") arr[pos] = hexChars[Math.floor(Math.random() * 16)];
                    hexBlocks[i] = arr.join("");
                }
                ctx.fillStyle = `hsla(200, 50%, 50%, ${alpha * 0.12})`;
                ctx.fillText(hexBlocks[i], hx, hy + i * 12);
            }

            // Security labels — top left area
            ctx.font = "600 10px 'Inter', monospace";
            const lx = cx - r * 1.3;
            const ly = cy - r * 0.5;
            for (let i = 0; i < 4; i++) {
                const flicker = 0.7 + 0.3 * Math.sin(time * 1.5 + i * 2);
                ctx.fillStyle = `hsla(200, 60%, 50%, ${alpha * 0.15 * flicker})`;
                ctx.fillText(labels[i], lx, ly + i * 18);
            }

            // More labels — bottom left
            for (let i = 4; i < labels.length; i++) {
                const flicker = 0.7 + 0.3 * Math.sin(time * 1.2 + i * 3);
                ctx.fillStyle = `hsla(200, 60%, 50%, ${alpha * 0.12 * flicker})`;
                ctx.fillText(labels[i], lx, cy + r * 0.4 + (i - 4) * 18);
            }

            // Scanning line on globe
            const scanAngle = time * 0.8 % PI2;
            const scanX = cx + Math.cos(scanAngle) * r * 1.1;
            const scanY = cy + Math.sin(scanAngle) * r * 1.1;
            ctx.strokeStyle = `hsla(200, 70%, 55%, ${alpha * 0.08})`;
            ctx.lineWidth = 0.5;
            ctx.beginPath();
            ctx.moveTo(cx, cy);
            ctx.lineTo(scanX, scanY);
            ctx.stroke();

            // Corner brackets
            const br = r * 1.15;
            const bLen = 20;
            ctx.strokeStyle = `hsla(200, 50%, 50%, ${alpha * 0.2})`;
            ctx.lineWidth = 1;
            // Top-left
            ctx.beginPath();
            ctx.moveTo(cx - br, cy - br + bLen); ctx.lineTo(cx - br, cy - br); ctx.lineTo(cx - br + bLen, cy - br);
            ctx.stroke();
            // Top-right
            ctx.beginPath();
            ctx.moveTo(cx + br - bLen, cy - br); ctx.lineTo(cx + br, cy - br); ctx.lineTo(cx + br, cy - br + bLen);
            ctx.stroke();
            // Bottom-left
            ctx.beginPath();
            ctx.moveTo(cx - br, cy + br - bLen); ctx.lineTo(cx - br, cy + br); ctx.lineTo(cx - br + bLen, cy + br);
            ctx.stroke();
            // Bottom-right
            ctx.beginPath();
            ctx.moveTo(cx + br - bLen, cy + br); ctx.lineTo(cx + br, cy + br); ctx.lineTo(cx + br, cy + br - bLen);
            ctx.stroke();

            // Crosshair at center
            ctx.strokeStyle = `hsla(200, 50%, 50%, ${alpha * 0.08})`;
            ctx.lineWidth = 0.5;
            ctx.beginPath();
            ctx.moveTo(cx - 15, cy); ctx.lineTo(cx + 15, cy);
            ctx.moveTo(cx, cy - 15); ctx.lineTo(cx, cy + 15);
            ctx.stroke();
        }

        function draw() {
            rotation += 0.003;

            // Smoothly lerp tilt toward mouse position
            const targetTiltX = (my - 0.5) * 0.5;
            const targetTiltY = (mx - 0.5) * 0.5;
            tiltX += (targetTiltX - tiltX) * 0.03;
            tiltY += (targetTiltY - tiltY) * 0.03;

            ctx.clearRect(0, 0, canvasEl.width, canvasEl.height);

            // Globe positioned to far right, vertically centered
            const cx = canvasEl.width * 0.82;
            const cy = canvasEl.height * 0.45;
            const r = Math.min(canvasEl.width, canvasEl.height) * 0.25;

            const alpha = 0.9; // Always visible, subtle

            drawGlobe(cx, cy, r, alpha);
            drawHUD(cx, cy, r, alpha);

            animId = requestAnimationFrame(draw);
        }

        draw();

        return {
            destroy() {
                cancelAnimationFrame(animId);
                window.removeEventListener("resize", resize);
            }
        };
    }
</script>

<svelte:head>
    <title
        >Zohar Global Security | Executive Protection for Family Offices & UHNW
        Families</title
    >
    <meta
        name="description"
        content="Zohar Global Security provides full-spectrum executive protection, risk assessment, and security management for family offices, ultra-high net worth individuals, and Fortune 500 executives. 40 years in corporate and government security. Absolute discretion."
    />
    <link rel="canonical" href="https://zoharsecurity.com/" />

    <!-- Open Graph -->
    <meta property="og:type" content="website" />
    <meta
        property="og:title"
        content="Zohar Global Security | Executive Protection for Family Offices & UHNW Families"
    />
    <meta
        property="og:description"
        content="Private security built on trust, discretion, and 40 years in corporate and government security. Full-spectrum protection for family offices, UHNW individuals, and Fortune 500 executives."
    />
    <meta property="og:url" content="https://zoharsecurity.com/" />
    <meta property="og:site_name" content="Zohar Global Security" />
    <meta property="og:locale" content="en_US" />

    <!-- Open Graph Image -->
    <meta property="og:image" content="https://zoharsecurity.com/og-image.jpg" />

    <!-- Twitter Card -->
    <meta name="twitter:card" content="summary_large_image" />
    <meta
        name="twitter:title"
        content="Zohar Global Security | Executive Protection"
    />
    <meta
        name="twitter:description"
        content="Private security built on trust, discretion, and 40 years in corporate and government security. Full-spectrum protection for family offices, UHNW individuals, and Fortune 500 executives."
    />
    <meta name="twitter:image" content="https://zoharsecurity.com/og-image.jpg" />

    <!-- JSON-LD: Organization -->
    {@html `<script type="application/ld+json">${JSON.stringify({
        "@context": "https://schema.org",
        "@type": "Organization",
        name: "Zohar Global Security",
        url: "https://zoharsecurity.com",
        description:
            "Full-spectrum executive protection and security management for family offices, ultra-high net worth individuals, and Fortune 500 executives.",
        telephone: "+1-702-743-0031",
        email: "info@zoharsecurity.com",
        founder: {
            "@type": "Person",
            name: "Zohar Lahav",
            jobTitle: "Founder",
            description:
                "40-year veteran of corporate, executive, and government security. Former Israeli Secret Service Special Agent. Extensive experience securing high-net-worth individuals, global-leaders, and Fortune 500 executives.",
        },
        serviceType: [
            "Executive Protection",
            "Risk Assessment",
            "Security Consulting",
            "Outsourced Chief Security Officer",
            "Travel Security",
            "Background Checks",
            "Cyber Protection",
            "Investigations and Surveillance",
        ],
    })}</script>`}
    {@html `<script type="application/ld+json">${JSON.stringify({
        "@context": "https://schema.org",
        "@type": "FAQPage",
        mainEntity: [
            {
                "@type": "Question",
                name: "How do I know your people won't become the threat?",
                acceptedAnswer: {
                    "@type": "Answer",
                    text: "Every operator we place undergoes the same level of vetting we apply to the people surrounding you — polygraph-level screening, financial background analysis, and continuous monitoring throughout the engagement. We rotate and compartmentalize information so no single agent holds a complete picture of your family's patterns, assets, or vulnerabilities. The insider threat is the first thing we design against, not an afterthought.",
                },
            },
            {
                "@type": "Question",
                name: "My children already resist the security we have. How do you keep protection from damaging their sense of a normal life?",
                acceptedAnswer: {
                    "@type": "Answer",
                    text: 'Children adapt to what feels normal, and the problem is almost always visibility, not presence. We train our agents to operate in roles and postures that do not signal "security" to a child or their peers. When it is done correctly, your kids do not grow up feeling watched. They grow up feeling safe without knowing why.',
                },
            },
            {
                "@type": "Question",
                name: "We already have security in place. How do I know it's actually not working?",
                acceptedAnswer: {
                    "@type": "Answer",
                    text: "Most legacy security programs were built for a threat environment that no longer exists, or they were built by someone who confused presence with protection. We conduct a confidential, zero-disruption audit of your current program — access protocols, personnel reliability, route predictability, digital exposure — and show you exactly where the gaps are. If there are none, we will tell you that too.",
                },
            },
            {
                "@type": "Question",
                name: "What happens if something actually goes wrong — a real emergency, not a drill?",
                acceptedAnswer: {
                    "@type": "Answer",
                    text: "Our crisis protocols are built by people who have managed real incidents for heads of state and billionaire families — not tabletop exercises. Every principal we protect has a pre-staged extraction plan, a communication protocol that works when phones do not, and a designated team leader who has authority to act without waiting for approval. You will not be calling a hotline. You will be talking to someone who already knows your family, your locations, and what to do next.",
                },
            },
            {
                "@type": "Question",
                name: "What happens during the transition if we leave our current provider and move to you?",
                acceptedAnswer: {
                    "@type": "Answer",
                    text: "The transition window is when you are most exposed, and we plan for it accordingly. Before your current provider is notified, we have already mapped your existing coverage, identified the gaps that will open, and positioned our people to close them. There is no moment where your family is unprotected. We treat the handoff as an operation, not an onboarding process.",
                },
            },
            {
                "@type": "Question",
                name: "What if I have a concern you haven't covered here?",
                acceptedAnswer: {
                    "@type": "Answer",
                    text: "We expect that. Every family's situation is different, and the questions that matter most are usually the ones that are hardest to ask publicly. We are confident we can address whatever is on your mind — whether it involves a specific threat, a personal circumstance, or something you have never told another security provider. Call or write. The conversation is confidential, and there is no concern too sensitive to raise.",
                },
            },
        ],
    })}</script>`}
</svelte:head>

<nav class="site-nav">
    <div class="nav-inner">
        <a href="#" class="nav-brand">
            <img src="/logo.png" alt="" class="nav-logo" />
            <span>Zohar Global Security</span>
        </a>
        <div class="nav-links">
            <a href="#who-we-protect">Protection</a>
            <a href="#services">Services</a>
            <a href="#about">Founder</a>
            <a href="#faq">FAQ</a>
            <a href="#contact" class="nav-cta">Contact</a>
        </div>
    </div>
</nav>

<header>
    <canvas use:initCanvas class="header-canvas" aria-hidden="true"></canvas>
    <div class="container">
        <div class="hero-grid">
            <div class="hero-text">
                <h1 tabindex="0">
                    Securing What Matters Most: <br /> Family, Business, and Legacy
                </h1>
                <p class="tagline">
                    40 Years Service Built on Trust,
                    Discretion, and Bespoke Solutions
                </p>
                <p class="intro">
                    Full-spectrum security management for principals and their
                    families — executive protection, risk assessment, travel
                    security, and crisis response. Founded by Zohar Lahav, a former
                    Israeli Secret Service Special Agent with extensive
                    experience in securing billionaires, world leaders, and
                    Fortune 500 executives.
                </p>
                <p class="cta-label">
                    For an initial consultation, fee schedule, or references.
                </p>
                <div class="hero-contact">
                    <a href="tel:+17027430031">702-743-0031</a>
                    <span class="divider" aria-hidden="true">|</span>
                    <a href="mailto:info@zoharsecurity.com"
                        >info@zoharsecurity.com</a
                    >
                    <span class="divider" aria-hidden="true">|</span>
                    <a href="/research"
                        >The definitive reading list for UHNW security
                        leadership</a
                    >
                </div>
                <a href="mailto:info@zoharsecurity.com?subject=Consultation%20Request" class="cta-button">Schedule a Consultation</a>
                <p class="cta-assurance">Prompt response. Confidential. No obligation.</p>
            </div>
        </div>
    </div>
</header>

<main id="main">
    <section aria-labelledby="who-we-protect" class="bg-light">
        <div class="container">
            <h2 id="who-we-protect" tabindex="0">Who We Protect</h2>

            <div class="card-grid">
                <article>
                    <span class="card-number" aria-hidden="true">01</span>
                    <h3>Family Offices</h3>
                    <p>
                        We secure the interests, assets, and daily lives of the
                        individuals who run single and multi-family offices.
                        From residences to travel to public appearances, we
                        manage risk so you don't have to think about it.
                    </p>
                </article>

                <article>
                    <span class="card-number" aria-hidden="true">02</span>
                    <h3>UHNW Individuals</h3>
                    <p>
                        High visibility creates exposure. We build protection
                        programs around your life as it actually moves. Not
                        templates. Not guesswork. Programs shaped by how you
                        live, work, and travel.
                    </p>
                </article>

                <article>
                    <span class="card-number" aria-hidden="true">03</span>
                    <h3>Top Executives</h3>
                    <p>
                        We protect corporate leaders of Fortune 500 companies and their families from
                        threats that extend beyond the office. Board members,
                        C-suite principals, and their households receive
                        integrated security that covers every environment.
                    </p>
                </article>
            </div>
        </div>
    </section>

    <section aria-labelledby="services">
        <div class="container">
            <h2 id="services" tabindex="0">Our Services</h2>

            <!-- Hero Services — top 3 -->
            <div class="services-hero">
                <article>
                    <h3>Risk Assessment and Security Consulting</h3>
                    <p>
                        Every engagement starts here. We study your routines,
                        movement patterns, existing security systems, and the
                        environments your family operates in. You receive a
                        clear risk analysis and a security program built around
                        what we find. Not what we assume.
                    </p>
                </article>

                <article>
                    <h3>Personal and Executive Protection</h3>
                    <p>
                        Trained protection agents embedded in your daily life.
                        Visible when needed. Invisible when not. Our agents come
                        from military, government, and elite law enforcement
                        backgrounds.
                    </p>
                    <ul class="protection-services">
                        <li>
                            <strong>Travel Security</strong> &mdash; Domestic or international.
                            Commercial or private aviation. Armored ground transport
                            or maritime. We plan the route, assess the risk, and place
                            the right people at every point of transit.
                        </li>
                        <li>
                            <strong>School and Campus Safety</strong> &mdash; We assess
                            the physical layout, entry points, and emergency protocols
                            of every institution your children attend. Then we close
                            the gaps.
                        </li>
                        <li>
                            <strong
                                >Background Checks for Household and Executive
                                Staff</strong
                            > &mdash; Nannies, drivers, personal assistants, estate
                            managers, new hires. We verify who is in proximity to
                            your family before they get close.
                        </li>
                        <li>
                            <strong
                                >Cyber Protection and Forensic Data Services</strong
                            > &mdash; Digital exposure is physical exposure. We protect
                            your communications, devices, and data from surveillance,
                            breaches, and exploitation.
                        </li>
                        <li>
                            <strong
                                >Investigations, Surveillance, and Counter
                                Surveillance</strong
                            > &mdash; When something feels wrong, we find out what
                            it is. Discreetly. Our team conducts investigations, deploys
                            surveillance, and detects if you are being watched or
                            followed.
                        </li>
                        <li>
                            <strong>Custom Security Plans</strong> &mdash; We can
                            customize a security plan to the exact needs of your organization.
                        </li>
                    </ul>
                </article>

                <article>
                    <h3>Outsourced Chief Security Officer</h3>
                    <p>
                        A full-time CSO without the full-time cost. We place a
                        senior security leader inside your organization to
                        build, manage, and run your entire protection program.
                        One monthly fee covers the role, the administration, and
                        the infrastructure.
                    </p>
                    <ul class="cso-benefits">
                        <li>
                            <strong>No Hiring Process</strong> &mdash; We place a
                            proven security leader from our team immediately. Vetted.
                            Experienced. Ready.
                        </li>
                        <li>
                            <strong>One Predictable Fee</strong> &mdash; Monthly or
                            annual. No salary negotiations. No benefits packages.
                            No relocation costs. One fee for the role and everything
                            that supports it.
                        </li>
                        <li>
                            <strong
                                >No Office Overhead or Administrative Burden</strong
                            > &mdash; No furniture, no tech setup, no support staff
                            to manage. We handle our own operations so your organization
                            stays lean.
                        </li>
                    </ul>
                </article>
            </div>
        </div>
    </section>

    <section aria-labelledby="about" class="bg-light">
        <div class="container">
            <h2 id="about" tabindex="0">About Zohar Lahav</h2>
            <p class="section-intro">
                Zohar Lahav has spent 40 years in corporate, executive, and
                government security. He built Zohar Global Security on a career
                that most people in this industry cannot match.
            </p>

            <div class="about-layout">
                <img
                    src="/zohar-hs.webp"
                    alt="Zohar Lahav, founder of Zohar Global Security"
                    class="founder-photo"
                    width="360"
                    height="450"
                    loading="lazy"
                />

                <div class="timeline">
                    <article>
                        <h3>
                            Israeli Defense Forces, Elite Paratroopers Commando
                        </h3>
                        <p>
                            Lahav began in 1983 serving in an elite Paratroopers
                            Commando and Reconnaissance Unit within the Israeli
                            Defense Forces.
                        </p>
                    </article>

                    <article>
                        <h3>Israeli Secret Service, Special Agent</h3>
                        <p>
                            He served as a Special Agent protecting the
                            President of Israel, the Prime Minister, cabinet
                            ministers, and visiting foreign officials.
                        </p>
                    </article>

                    <article>
                        <h3>25 Years Protecting a Billionaire Family Office</h3>
                        <p>
                            Lahav served as Vice President of Executive
                            Protection and Head of Security for a family office
                            and its related businesses for 25 years. He
                            protected the CEO, COO, and their families. He built
                            and ran programs spanning aviation, maritime,
                            transportation, physical and electronic
                            surveillance, and intelligence operations worldwide.
                        </p>
                    </article>

                    <article>
                        <h3>Head of Security, Miami Mayor's Office</h3>
                        <p>
                            Lahav served as Sergeant at Arms for the City of
                            Miami, protecting the Mayor, City Commissioners, and
                            selected delegates. He also worked with the
                            Government of Israel's Foreign Ministry, Israeli
                            Airlines EL AL, and coordinated with U.S. federal
                            and local law enforcement.
                        </p>
                    </article>
                </div>
            </div>
        </div>
    </section>

    <section aria-labelledby="faq" class="faq-section">
        <div class="container">
            <h2 id="faq" tabindex="0">Frequently Asked Questions</h2>

            <div class="faq-list">
                <details>
                    <summary
                        >How do I know your people won't become the threat?</summary
                    >
                    <p>
                        Every operator we place undergoes the same level of
                        vetting we apply to the people surrounding you —
                        polygraph-level screening, financial background
                        analysis, and continuous monitoring throughout the
                        engagement. We rotate and compartmentalize information
                        so no single agent holds a complete picture of your
                        family's patterns, assets, or vulnerabilities. The
                        insider threat is the first thing we design against, not
                        an afterthought.
                    </p>
                </details>

                <details>
                    <summary
                        >My children already resist the security we have. How do
                        you keep protection from damaging their sense of a
                        normal life?</summary
                    >
                    <p>
                        Children adapt to what feels normal, and the problem is
                        almost always visibility, not presence. We train our
                        agents to operate in roles and postures that do not
                        signal "security" to a child or their peers. When it is
                        done correctly, your kids do not grow up feeling
                        watched. They grow up feeling safe without knowing why.
                    </p>
                </details>

                <details>
                    <summary
                        >We already have security in place. How do I know it's
                        actually not working?</summary
                    >
                    <p>
                        Most legacy security programs were built for a threat
                        environment that no longer exists, or they were built by
                        someone who confused presence with protection. We
                        conduct a confidential, zero-disruption audit of your
                        current program — access protocols, personnel
                        reliability, route predictability, digital exposure —
                        and show you exactly where the gaps are. If there are
                        none, we will tell you that too.
                    </p>
                </details>

                <details>
                    <summary
                        >What happens if something actually goes wrong — a real
                        emergency, not a drill?</summary
                    >
                    <p>
                        Our crisis protocols are built by people who have
                        managed real incidents for heads of state and
                        billionaire families — not tabletop exercises. Every
                        principal we protect has a pre-staged extraction plan, a
                        communication protocol that works when phones do not,
                        and a designated team leader who has authority to act
                        without waiting for approval. You will not be calling a
                        hotline. You will be talking to someone who already
                        knows your family, your locations, and what to do next.
                    </p>
                </details>

                <details>
                    <summary
                        >What happens during the transition if we leave our
                        current provider and move to you?</summary
                    >
                    <p>
                        The transition window is when you are most exposed, and
                        we plan for it accordingly. Before your current provider
                        is notified, we have already mapped your existing
                        coverage, identified the gaps that will open, and
                        positioned our people to close them. There is no moment
                        where your family is unprotected. We treat the handoff
                        as an operation, not an onboarding process.
                    </p>
                </details>

                <details>
                    <summary
                        >What if I have a concern you haven't covered here?</summary
                    >
                    <p>
                        We expect that. Every family's situation is different,
                        and the questions that matter most are usually the ones
                        that are hardest to ask publicly. We are confident we
                        can address whatever is on your mind — whether it
                        involves a specific threat, a personal circumstance, or
                        something you have never told another security provider.
                        Call or write. The conversation is confidential, and
                        there is no concern too sensitive to raise.
                    </p>
                </details>
            </div>
        </div>
    </section>
</main>

<footer id="contact" class="bg-light">
    <div class="container">
        <h2 tabindex="0">Begin a Confidential Conversation</h2>
        <p class="section-intro">
            Every engagement starts with a private discussion. No obligation, no pressure — just a direct conversation about what matters to you and your family.
        </p>
        <div class="contact-links">
            <a href="tel:+17027430031">702-743-0031</a>
            <span class="divider" aria-hidden="true">|</span>
            <a href="mailto:info@zoharsecurity.com">info@zoharsecurity.com</a>
            <span class="divider" aria-hidden="true">|</span>
            <a href="/research"
                >The definitive reading list for UHNW security leadership</a
            >
        </div>
        <p class="copyright">
            &copy; {new Date().getFullYear()} Zohar Global Security. All rights reserved.
        </p>
        <p class="legal">ZGS LLC &middot; 5013 S Louise Ave #1521, Sioux Falls, SD 57108</p>
    </div>
</footer>

<style>
    /* ── Navigation ─────────────────────────── */
    .site-nav {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        z-index: 1000;
        background: hsla(200, 20%, 5%, 0.85);
        backdrop-filter: blur(12px);
        -webkit-backdrop-filter: blur(12px);
        border-bottom: 1px solid hsla(200, 10%, 30%, 0.3);
        padding: 0 var(--spacing-md);
    }

    .nav-inner {
        max-width: var(--max-width-container);
        margin: 0 auto;
        display: flex;
        align-items: center;
        justify-content: space-between;
        height: 52px;
    }

    .nav-brand {
        display: flex;
        align-items: center;
        gap: 10px;
        font-size: 11px;
        font-weight: 500;
        letter-spacing: 0.2em;
        text-transform: uppercase;
        color: var(--c-accent);
        text-decoration: none;
    }

    .nav-logo {
        width: 22px;
        height: auto;
    }

    .nav-links {
        display: flex;
        align-items: center;
        gap: var(--spacing-md);
    }

    .nav-links a {
        font-size: 12px;
        font-weight: 500;
        letter-spacing: 0.06em;
        color: var(--c-text-muted);
        text-decoration: none;
        text-transform: uppercase;
        transition: color 0.2s;
    }

    .nav-links a:hover {
        color: var(--c-heading);
        opacity: 1;
    }

    .nav-cta {
        color: var(--c-heading) !important;
        border: 1px solid hsla(200, 50%, 50%, 0.4);
        padding: 6px 14px;
        border-radius: 2px;
        transition: all 0.2s !important;
    }

    .nav-cta:hover {
        background: hsla(200, 50%, 50%, 0.15);
        border-color: hsla(200, 50%, 50%, 0.6);
    }

    @media (max-width: 640px) {
        .nav-links a:not(.nav-cta) {
            display: none;
        }
    }

    /* ── CTA Button ─────────────────────────── */
    .cta-button {
        display: inline-block;
        margin-top: var(--spacing-lg);
        padding: 14px 32px;
        background: hsl(200, 50%, 50%);
        color: white !important;
        font-size: 14px;
        font-weight: 600;
        letter-spacing: 0.08em;
        text-transform: uppercase;
        text-decoration: none;
        border-radius: 2px;
        transition: all 0.2s;
    }

    .cta-button:hover {
        background: hsl(200, 55%, 45%);
        opacity: 1 !important;
        transform: translateY(-1px);
    }

    /* ── Layout ─────────────────────────────── */
    .container {
        max-width: var(--max-width-container);
        margin: 0 auto;
        padding: 0 var(--spacing-md);
        position: relative;
        z-index: 1;
    }

    /* ── Header ─────────────────────────────── */
    header {
        position: relative;
        padding: 64px 0 var(--spacing-3xl);
        padding-top: 100px;
        border-bottom: var(--border-width) solid var(--c-mute);
        min-height: 90vh;
        display: flex;
        flex-direction: column;
        justify-content: center;
        overflow: hidden;
    }

    .header-canvas {
        position: absolute;
        inset: 0;
        width: 100%;
        height: 100%;
        pointer-events: none;
    }

    @media (max-width: 768px) {
        .header-canvas {
            display: none;
        }
    }

    .brand {
        display: flex;
        align-items: center;
        gap: 12px;
        margin-bottom: var(--spacing-xl);
    }

    .brand-logo {
        width: 36px;
        height: auto;
    }

    .brand-name {
        font-size: 13px;
        text-transform: uppercase;
        letter-spacing: 0.2em;
        color: var(--c-accent);
        font-weight: 500;
    }

    .hero-grid {
        display: block;
        padding-bottom: var(--spacing-2xl);
    }

    h1 {
        font-size: var(--font-size-h1);
        font-weight: 800;
        letter-spacing: -0.03em;
        color: var(--c-heading);
        line-height: var(--line-height-tight);
    }

    .tagline {
        margin-top: var(--spacing-md);
        font-size: 20px;
        color: var(--c-text-muted);
        font-weight: 400;
        letter-spacing: 0.01em;
    }

    .intro {
        margin-top: var(--spacing-md);
        max-width: var(--max-width-prose);
        font-size: var(--font-size-base);
        line-height: var(--line-height-relaxed);
    }

    .cta-label {
        margin-top: var(--spacing-xl);
        font-size: 13px;
        text-transform: uppercase;
        letter-spacing: 0.1em;
        color: var(--c-text-muted);
    }

    .hero-contact {
        display: flex;
        align-items: center;
        gap: var(--spacing-sm);
        margin-top: var(--spacing-xs);
        font-size: var(--font-size-base);
        flex-wrap: wrap;
    }

    .hero-contact a {
        font-weight: 500;
    }

    .cta-assurance {
        margin-top: var(--spacing-sm);
        font-size: 13px;
        color: var(--c-text-muted);
        letter-spacing: 0.02em;
    }

    /* ── Sections ───────────────────────────── */
    section {
        padding: var(--spacing-4xl) 0;
        border-bottom: var(--border-width) solid var(--c-mute);
    }

    h2 {
        font-size: var(--font-size-h2);
        font-weight: 800;
        color: var(--c-heading);
        margin-bottom: var(--spacing-md);
        letter-spacing: -0.02em;
    }

    .section-intro {
        max-width: var(--max-width-prose);
        margin-bottom: var(--spacing-xl);
        line-height: var(--line-height-relaxed);
    }

    h3 {
        font-size: var(--font-size-h3);
        font-weight: 600;
        color: var(--c-heading);
        margin-bottom: var(--spacing-xs);
    }

    article p {
        line-height: var(--line-height-relaxed);
    }

    /* ── Card Grid (Who We Protect) ─────────── */
    .card-grid {
        display: grid;
        gap: var(--spacing-lg);
        margin-top: var(--spacing-xl);
    }

    .card-grid article {
        background: transparent;
        border: none;
        border-top: var(--border-width) solid var(--c-mute);
        border-radius: 0;
        padding: var(--spacing-lg) 0;
    }

    .card-number {
        display: block;
        font-size: 5rem;
        font-weight: 800;
        color: var(--c-accent);
        opacity: 0.12;
        line-height: 1;
        letter-spacing: -0.04em;
        margin-bottom: var(--spacing-sm);
    }

    /* ── Services — Hero Tier ──────────────── */
    .services-hero {
        display: grid;
        gap: 0;
        margin-top: var(--spacing-xl);
    }

    .services-hero article {
        background: transparent;
        border: none;
        border-left: 2px solid var(--c-accent);
        border-radius: 0;
        padding: var(--spacing-lg) 0 var(--spacing-lg) var(--spacing-lg);
        border-bottom: var(--border-width) solid var(--c-mute);
    }

    .services-hero article:last-child {
        border-bottom: none;
    }

    .services-hero article h3 {
        color: var(--c-accent);
    }

    .cso-benefits {
        list-style: none;
        margin-top: var(--spacing-md);
        padding-top: var(--spacing-md);
        border-top: var(--border-width) solid var(--c-mute);
        display: grid;
        gap: var(--spacing-sm);
    }

    .cso-benefits li {
        line-height: var(--line-height-relaxed);
        font-size: var(--font-size-base);
    }

    .cso-benefits strong {
        color: var(--c-heading);
    }

    /* ── Protection Services List ─────────── */
    .protection-services {
        list-style: none;
        margin-top: var(--spacing-md);
        padding-top: var(--spacing-md);
        border-top: var(--border-width) solid var(--c-mute);
        display: grid;
        gap: var(--spacing-sm);
    }

    .protection-services li {
        line-height: var(--line-height-relaxed);
        font-size: var(--font-size-base);
    }

    .protection-services strong {
        color: var(--c-heading);
    }

    /* ── About — Layout with Photo ──────────── */
    .about-layout {
        display: grid;
        gap: var(--spacing-xl);
        margin-top: var(--spacing-xl);
    }

    .founder-photo {
        width: 100%;
        max-width: 320px;
        height: auto;
        object-fit: cover;
        filter: grayscale(30%);
    }

    /* ── Timeline (About) ───────────────────── */
    .timeline {
        display: grid;
        gap: var(--spacing-xl);
        padding-left: var(--spacing-md);
        border-left: var(--border-width) solid var(--c-mute);
    }

    .timeline article {
        padding-left: var(--spacing-md);
    }

    /* ── FAQ ─────────────────────────────────── */
    .faq-section h2 {
        color: var(--c-heading);
    }

    .faq-list {
        margin-top: var(--spacing-xl);
        display: grid;
        gap: 0;
    }

    details {
        border-bottom: var(--border-width) solid var(--c-mute);
    }

    details summary {
        cursor: pointer;
        padding: var(--spacing-md) 0;
        font-weight: 600;
        color: var(--c-heading);
        font-size: 20px;
        list-style: none;
        display: flex;
        justify-content: space-between;
        align-items: center;
        gap: var(--spacing-sm);
    }

    details summary::after {
        content: "+";
        font-size: 1.25rem;
        color: var(--c-text-muted);
        flex-shrink: 0;
        transition: transform 0.2s ease;
    }

    details[open] summary::after {
        content: "\2212";
    }

    details summary::-webkit-details-marker {
        display: none;
    }

    details p {
        padding: 0 0 var(--spacing-md);
        line-height: var(--line-height-relaxed);
        max-width: var(--max-width-prose);
    }

    /* ── Footer ─────────────────────────────── */
    footer {
        padding: var(--spacing-3xl) 0 var(--spacing-2xl);
    }

    footer h2 {
        font-size: var(--font-size-h3);
    }

    .contact-links {
        display: flex;
        align-items: center;
        gap: var(--spacing-sm);
        margin-top: var(--spacing-xs);
        font-size: var(--font-size-base);
        flex-wrap: wrap;
    }

    .contact-links a {
        font-weight: 500;
    }

    .divider {
        color: var(--c-text-muted);
    }

    .copyright {
        margin-top: var(--spacing-2xl);
        font-size: var(--font-size-small);
        color: var(--c-text-muted);
    }

    .legal {
        margin-top: var(--spacing-xs);
        font-size: 11px;
        color: var(--c-text-muted);
        opacity: 0.6;
        letter-spacing: 0.02em;
    }

    /* ── Responsive ─────────────────────────── */
    @media (min-width: 640px) {
        .card-grid {
            grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
        }
    }

    @media (min-width: 768px) {
        header {
            padding: 0;
            min-height: 100vh;
        }

        h1 {
            font-size: clamp(3rem, 5vw, 4.5rem);
        }

        section {
            padding: var(--spacing-5xl) 0;
        }

        .about-layout {
            grid-template-columns: auto 1fr;
            align-items: start;
        }

        .founder-photo {
            max-width: 280px;
        }

        .card-number {
            font-size: 7rem;
        }
    }

    @media (max-width: 480px) {
        h1 {
            font-size: 2.2rem;
        }

        h2 {
            font-size: 1.5rem;
        }

        header {
            padding: var(--spacing-2xl) 0 var(--spacing-xl);
        }

        section {
            padding: var(--spacing-3xl) 0;
        }

        .container {
            padding: 0 var(--spacing-sm);
        }
    }
</style>
