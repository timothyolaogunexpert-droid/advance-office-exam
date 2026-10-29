Advance Office Examination Portal
Production-oriented Netlify + Supabase exam application with 75 questions worth 100 marks (50 multiple choice × 1, 20 short answer × 1, 5 practical × 6), 60-minute timer, server-side objective grading, manual written grading, admin-only results, presence monitoring, warnings and email alerts.
Deploy (required steps)
1. Create a Supabase project. In its SQL Editor, run the complete schema.sql. Confirm the attempts and events tables exist.
2. Create a Resend account, verify a sending domain, and obtain an API key. Set EMAIL_FROM to an address at that verified domain. The administrator notification inbox is preconfigured as mattvictorexpert@gmail.com (ADMIN_EMAIL). You still need a verified sending domain and a Resend API key for email delivery.
3. Push this directory to a GitHub repository and import the repository into Netlify (recommended). Netlify will read netlify.toml, run npm run build, publish dist, and deploy netlify/functions/api.js. Alternatively run npm install && npm run build and deploy with Netlify CLI, ensuring functions are included. Do not use simple drag-and-drop of dist: it omits server functions.
4. In Netlify Site configuration → Environment variables, add all keys in .env.example, replacing sample values. SUPABASE_SERVICE_ROLE_KEY is the secret service-role key from Supabase project settings; never put it in frontend files. ADMIN_PASSWORD should be a strong unique secret. Redeploy after adding variables.
5. Visit the Netlify URL, register a test student, answer and submit. Then select Administrator login at the bottom of the homepage. Check live monitoring, results, grading, and email delivery.
6. For local development, copy .env.example to .env, fill secrets, run npm install, then npx netlify dev. Keep .env out of version control.
Admin workflow
- The admin dashboard is password protected by a Netlify Function. The password is held in the current browser tab memory, not stored in localStorage.
- The dashboard refreshes every 10 seconds. Online means an active attempt with a heartbeat in the last 45 seconds; it is an estimate, not a perfect count of people viewing the site. Students not in an active exam are not counted.
- Select View / Grade to inspect written responses, review violation events, and assign marks (0–1 for short answers, 0–6 for practical responses, increments of 0.5). Grading is saved and can be edited later. Students never see scores.
- Export CSV from the dashboard for results and monitoring data. Resend sends the administrator a submission notification with objective score; manual grading happens later.
Security and operational limitations
- Browsers cannot reliably prevent switching apps, using another device, disconnecting, or closing a window. Fullscreen is voluntary and can be exited. Visibility, blur, and fullscreen violations are best-effort signals and may include false positives (especially on mobile); the 3-warning rule is strict. Test thoroughly with your students' devices.
- The server controls the 60-minute deadline, answer persistence, final scoring, and student identity uniqueness. A student may resume an existing session only on the same browser/device while the token remains in localStorage. Re-registering with the same email or student ID is blocked. Administrators must manually delete an attempt in Supabase to authorize a retake.
- Database tables have row-level security enabled with no public access policies. Only server functions use the Supabase service role. Never expose secrets in client code.
- The admin password uses a single shared secret. For a large institution, replace it with role-based admin accounts, MFA, rate limiting, and audit logging. Add bot protection/rate limits to the start and admin endpoints before public high-volume launch.
- Presence uses polling/heartbeats, not WebSockets; near-real-time counts may lag by up to ~45 seconds. The dashboard lists at most the latest 1,000 attempts.
- Written answers are not automatically evaluated. Admin emails report objective scores only until manual grading.
- A browser crash, network outage, or failed email delivery can prevent instant reporting; responses saved before the outage remain in Supabase. If an exam expires while offline, its state is marked expired when the student next contacts the server. For automatic offline expiry processing, add a scheduled server job.
- This app is a practical deployment starter, not a formally audited proctoring system. Test exam flows and accessibility before real assessments.
Configuration
See .env.example. Do not commit real credentials.
October 2026 question bank update
The 75 questions in questions.js now match the approved Advance Office examination: 50 balanced-option MCQs, 20 short responses, and 5 practical scenarios. The confidential marking guide is in ADMIN_MARKING_GUIDE.md at the project root, outside the published dist directory. Do not place the marking guide in public or dist. The server-side answer key is never sent to student clients.

Before admitting students, verify the 50 MCQ answer positions and review all written rubrics. Existing attempts created with a previous question bank should be completed or cleared before deployment because this version uses the same question IDs with revised content. Test the entire flow on a staging deployment and verify that submitted answers, warnings, and scores are recorded correctly.
Administrator email login and FormSubmit notifications
The admin login now requires BOTH mattvictorexpert@gmail.com (or your configured ADMIN_EMAIL) and the ADMIN_PASSWORD secret. The password is configured as a Netlify environment variable and must never be included in browser files. This is a shared-secret admin login, not a full identity provider with MFA; for production with multiple administrators, use a proper identity provider.

Set EMAIL_PROVIDER=formsubmit and ADMIN_EMAIL=mattvictorexpert@gmail.com in Netlify environment variables. The server-side function sends submission notification summaries to https://formsubmit.co/ajax/mattvictorexpert@gmail.com; students never contact FormSubmit directly. FormSubmit requires activation/confirmation by the mailbox owner before messages are delivered. Complete its confirmation email and send a test submission. FormSubmit is a third-party service; do not use it to send confidential full written answers or passwords. Written answers remain in Supabase for admin review. You can instead use verified-domain Resend by setting EMAIL_PROVIDER=resend, RESEND_API_KEY, and EMAIL_FROM.

Deployment cannot be completed by the ZIP alone: create the Supabase project, run schema.sql, configure environment variables in Netlify, deploy, and test with a student and an admin account. The email address is not a substitute for a secure admin password.
Separate HTML pages
- public/index.html / /index.html — student registration and examination.
- public/admin.html / /admin.html — private administrator login/dashboard.
- Both use public/app.js, public/style.css, and secure /api/* Netlify functions. Upload the complete repository using Netlify Git deployment; static HTML files alone cannot provide database monitoring or secure scoring.
