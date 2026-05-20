# 🏥 System Design Interview Questions — Prescripto Doctor Appointment System

> **60 curated system design questions** covering architecture, database, security, scalability, DevOps, and real-time features of the Prescripto platform.

---

## 📋 Table of Contents

1. [High-Level Architecture](#-1-high-level-architecture)
2. [Database Design](#-2-database-design)
3. [Authentication & Security](#-3-authentication--security)
4. [Appointment Booking Engine](#-4-appointment-booking-engine)
5. [Payment Gateway](#-5-payment-gateway)
6. [File Upload & Storage](#-6-file-upload--storage)
7. [API Design & REST](#-7-api-design--rest)
8. [DevOps, Docker & CI/CD](#-8-devops-docker--cicd)
9. [Scalability & Performance](#-9-scalability--performance)
10. [Real-World Production Scenarios](#-10-real-world-production-scenarios)

---

## 🏗️ 1. High-Level Architecture

**Q1. Draw and explain the high-level architecture of Prescripto.**
> The system has three React frontends (Patient, Admin, Doctor), a single Express.js backend API, MongoDB Atlas as the database, Cloudinary for image storage, and Razorpay for payments. They are containerised with Docker and deployed on Render via a Jenkins CI/CD pipeline.

**Q2. Why did you split the frontend into three separate React apps (Patient, Admin, Doctor) instead of one app with role-based routing?**
> Separation of concerns. Each portal has a completely different set of pages, components, and business logic. Splitting them reduces bundle size for each user, improves security isolation (Admin code never ships to patients), and allows independent deployments.

**Q3. What is a monorepo and is Prescripto structured as one?**
> A monorepo stores multiple related projects in a single Git repository. Yes — `frontend/`, `admin/`, and `backend/` all live inside one repo, sharing a single `Jenkinsfile` and `docker-compose.yml`, which simplifies CI/CD orchestration.

**Q4. What is the role of the Backend API in the three-portal architecture?**
> The backend is a single source of truth. All three portals communicate exclusively through the REST API. No portal writes directly to the database, ensuring all business logic, validation, and authorization is enforced centrally.

**Q5. How would you scale this architecture for 1 million concurrent users?**
> Horizontally scale the Node.js backend using a load balancer (e.g., AWS ALB) with multiple instances. Use Redis for shared session/cache, MongoDB Atlas auto-scaling for the database, and a CDN (Cloudfront) to serve React static assets globally.

**Q6. What is the difference between vertical and horizontal scaling? Which applies to your current Render deployment?**
> Vertical scaling means upgrading the server (more RAM/CPU). Horizontal scaling means adding more servers. Render free tier is vertical (single instance). Upgrading to paid instances with auto-scaling gives horizontal scaling.

---

## 🗄️ 2. Database Design

**Q7. Why did you choose MongoDB over PostgreSQL for this healthcare application?**
> MongoDB's flexible document schema suits varied doctor profiles and embedded appointment snapshots. The `slots_booked` object structure would require a complex junction table in SQL, whereas MongoDB stores it natively as an object.

**Q8. Explain the Embedded Document pattern used in the `appointments` collection.**
> Instead of only storing `userId` and `docId` references, the `userData` and `docData` objects are copied (embedded) into the appointment at the time of booking. This preserves historical accuracy — if a doctor raises their fees, old appointments still show the original amount.

**Q9. What are the three collections in Prescripto and how are they related?**
> `users` (patients), `doctors`, and `appointments`. One user has many appointments (1:N). One doctor has many appointments (1:N). The `appointments` collection is the join between them.

**Q10. Why is `slots_booked` an Object (Hash Map) instead of an Array?**
> Hash map lookup is O(1). Checking if a slot is available by date is simply `slots_booked["25_10_2023"]`. If it were an array of objects, we'd loop through all entries to find matching dates — O(N) complexity.

**Q11. What indexes would you add to improve query performance?**
> Unique index on `users.email` and `doctors.email` for fast login lookups. Compound index on `appointments.userId` and `appointments.docId` for dashboard queries. Index on `appointments.slotDate` for date-range scheduling queries.

**Q12. What is the `{ minimize: false }` option in Mongoose and why is it used in doctorSchema?**
> By default, Mongoose removes empty objects before saving to DB. `minimize: false` forces Mongoose to persist `slots_booked: {}` even when empty, so the key always exists and new dates can be added without needing `$set` to initialize it.

**Q13. How does `select('-password')` work and why is it important?**
> The `-password` prefix in Mongoose's `.select()` excludes the `password` field from query results. This prevents the hashed password from ever traveling over the network, reducing the attack surface even if a response is intercepted.

**Q14. How would you handle a doctor changing their consultation fee mid-booking?**
> The embedded document pattern already handles this. The `amount` and `docData` snapshot in each appointment record the fee at booking time, so any subsequent fee changes do not affect past appointments.

**Q15. What is a Race Condition and how can it affect the slot booking system?**
> Two users booking the same slot simultaneously. Both read `slots_booked` and see it as free, then both write their booking. MongoDB's atomic `$addToSet` operator prevents duplicates, or a distributed lock (Redis) can serialize access to the slot.

---

## 🔐 3. Authentication & Security

**Q16. Explain the three-token architecture (token, dtoken, atoken) in Prescripto.**
> Each portal sends its JWT in a different header key: patients use `token`, doctors use `dtoken`, admins use `atoken`. This enforces role isolation — a patient's token cannot be used on a doctor or admin route.

**Q17. What is JWT and how does `jwt.verify()` work?**
> JWT is a signed token with three parts: Header, Payload, Signature. `jwt.verify()` recomputes the HMAC signature using the `JWT_SECRET` and compares it with the token's signature. If they match, the token is authentic and unmodified.

**Q18. Why is the Admin JWT signed differently from Patient/Doctor JWTs?**
> Patient/Doctor JWTs contain `{ id: user._id }`. The Admin JWT is signed from the literal string `ADMIN_EMAIL + ADMIN_PASSWORD` stored in `.env`. The middleware decodes and compares this string directly, without a database lookup.

**Q19. What is Context Injection and why is it a security best practice?**
> After validating the JWT, the middleware extracts `req.userId = decoded.id` and attaches it to the request object. Controllers then use `req.userId` instead of trusting a `userId` from the request body, preventing IDOR (Insecure Direct Object Reference) attacks.

**Q20. What is the difference between Authentication and Authorization?**
> Authentication answers "Who are you?" (validating JWT). Authorization answers "What are you allowed to do?" (checking if a patient can access admin routes). Prescripto does both — auth middleware first verifies the token, then each controller enforces role-specific permissions.

**Q21. What is bcrypt's salt and what attack does it prevent?**
> A salt is random data appended to a password before hashing. It ensures two users with the same password have different hash values. This defeats Rainbow Table attacks, which rely on precomputed hash-to-password lookup tables.

**Q22. What is the risk of not setting `expiresIn` on your JWTs?**
> Tokens never expire. If a token is stolen from localStorage via XSS, the attacker has permanent access until the JWT_SECRET is rotated, which invalidates every user's session at once.

**Q23. What is XSS and how does it relate to localStorage token storage?**
> Cross-Site Scripting (XSS) is when malicious JavaScript is injected into your app. Since tokens are in `localStorage`, a single `localStorage.getItem('token')` call from injected script steals the token. `HttpOnly` cookies are immune because JS cannot read them.

**Q24. What is CSRF and are you vulnerable to it?**
> Cross-Site Request Forgery tricks the browser into making authenticated requests. Because Prescripto uses custom headers (`token`, `dtoken`, `atoken`) instead of automatic cookies, CSRF attacks cannot forge the headers and the system is largely immune.

**Q25. How would you implement Refresh Tokens for Prescripto?**
> Issue a short-lived Access Token (15 min) and a long-lived Refresh Token (7 days) stored in an `HttpOnly` cookie. When the access token expires, the frontend silently calls `/api/auth/refresh` to get a new access token without re-login.

---

## 📅 4. Appointment Booking Engine

**Q26. Walk through the complete appointment booking flow.**
> Patient selects a doctor → chooses date and time → frontend calls `POST /api/user/bookAppointment` → backend checks `slots_booked` for conflicts → adds `slotTime` to `slots_booked[slotDate]` on the doctor document → creates a new appointment document → returns success.

**Q27. How does your system generate available time slots?**
> The frontend generates 7 days of slots (e.g., 10:00 AM to 9:00 PM in 30-min intervals) and compares each slot against `doctor.slots_booked[date]`. Slots already in the array are rendered as disabled/greyed out.

**Q28. What happens when a patient cancels an appointment?**
> The `cancelled` flag on the appointment document is set to `true`. Simultaneously, the cancelled `slotTime` is removed from `doctor.slots_booked[slotDate]` using `$pull`, freeing the slot for another patient to book.

**Q29. How does the Doctor mark an appointment as completed?**
> The Doctor's frontend calls `POST /api/doctor/completeAppointment` with the appointment ID. The `authDoctor` middleware validates the token, and the controller sets `isCompleted: true` on the appointment document.

**Q30. What is the design flaw of storing `slotDate` as a string like "25_10_2023"?**
> String-based dates are not sortable or comparable with standard date operators. A better approach is to store `slotDate` as a native `Date` object or Unix timestamp, enabling efficient range queries like "show me all appointments this week".

---

## 💳 5. Payment Gateway

**Q31. Explain the two-step Razorpay payment flow.**
> Step 1 (Order Creation): Frontend calls backend, which calls Razorpay API to create an Order with the amount. Razorpay returns an `orderId`. Step 2 (Payment Verification): After the user pays, Razorpay sends back a signature. The backend verifies the signature using HMAC-SHA256 to confirm the payment is authentic.

**Q32. Why must payment verification happen on the backend, not the frontend?**
> The `RAZORPAY_KEY_SECRET` used to verify the signature must never be exposed to the browser. A hacker could fake a successful payment response on the frontend. Only the backend, with the secret, can cryptographically verify Razorpay's signature.

**Q33. What is HMAC-SHA256 and how is it used in payment verification?**
> HMAC is a message authentication algorithm. Razorpay computes `HMAC(orderId + "|" + paymentId, KEY_SECRET)`. Your backend recomputes it and compares. If they match, the payment is genuine and unmodified.

**Q34. What happens in the database after a successful payment?**
> The `payment` boolean field in the `appointments` collection is set to `true`, creating a permanent audit trail that this appointment was paid for.

**Q35. How would you handle a Razorpay webhook for delayed payment confirmations?**
> Set up a `POST /api/webhook/razorpay` endpoint. Razorpay posts a signed event payload when a payment status changes (even if the user closed the browser mid-payment). The backend verifies the signature and updates the `payment` flag accordingly.

---

## 🖼️ 6. File Upload & Storage

**Q36. Why did you use Cloudinary instead of storing images on the server's filesystem?**
> Server filesystem storage is ephemeral on platforms like Render (files are deleted on redeploy). Cloudinary provides persistent, CDN-backed cloud storage with automatic image optimization, resizing, and global delivery.

**Q37. What is Multer and what is its role in the upload flow?**
> Multer is an Express middleware for handling `multipart/form-data` (file uploads). It intercepts the incoming request, parses the binary file data, and makes it available as `req.file` before the controller runs.

**Q38. What is `memoryStorage` in Multer and why is it used for Cloudinary uploads?**
> `memoryStorage` keeps the uploaded file in RAM as a `Buffer` instead of writing it to disk. The Buffer is then piped directly to Cloudinary's upload stream, avoiding a temporary file write-then-read cycle.

**Q39. What file validation would you add to the upload endpoint?**
> Check `req.file.mimetype` against an allowlist (`image/jpeg`, `image/png`, `image/webp`). Also enforce a maximum file size via `multer({ limits: { fileSize: 5 * 1024 * 1024 } })`. Reject anything that doesn't meet these criteria.

**Q40. What is a Cloudinary Transform URL and how does it optimize performance?**
> Cloudinary URLs can include transformation parameters like `w_400,h_400,c_fill,q_auto,f_auto`. These resize, compress, and convert images to the most efficient format (e.g., WebP) on-the-fly, dramatically reducing page load times.

---

## 📡 7. API Design & REST

**Q41. What HTTP status codes should your API properly return?**
> `200 OK` for success, `201 Created` for new resources, `400 Bad Request` for validation errors, `401 Unauthorized` for missing/invalid tokens, `403 Forbidden` for insufficient permissions, `404 Not Found`, `409 Conflict` for duplicate emails, `500 Internal Server Error`.

**Q42. Your API currently returns `{ success: false }` with a 200 status on errors. What is the RESTful improvement?**
> Return proper HTTP status codes. A missing field should be `400`, a duplicate email should be `409 Conflict`, and an invalid token should be `401 Unauthorized`. This allows HTTP-aware infrastructure (reverse proxies, monitoring tools) to handle errors correctly.

**Q43. What is the difference between `PUT` and `PATCH`?**
> `PUT` replaces the entire resource. `PATCH` partially updates it. When a doctor updates their profile (only changing their `about` bio), `PATCH` is semantically correct. Most of Prescripto's "update" routes would be better as `PATCH`.

**Q44. What is API Versioning and how would you implement it for Prescripto?**
> Prefix routes with a version like `/api/v1/user/login`. If breaking changes are needed later, `/api/v2/user/login` can be added without breaking existing clients still using v1.

**Q45. How would you implement pagination for the `GET /api/admin/appointments` endpoint?**
> Accept `page` and `limit` query parameters. Use Mongoose's `.skip((page-1) * limit).limit(limit)` and return total count for frontend to calculate total pages: `{ data: [...], total: 500, page: 2, limit: 20 }`.

---

## 🐳 8. DevOps, Docker & CI/CD

**Q46. Explain the six stages of the Prescripto Jenkins pipeline.**
> (1) Checkout code from GitHub, (2) Create `.env` from Jenkins Credentials, (3) Build Docker images, (4) Deploy with docker-compose, (5) Health Check with `curl` on all three ports, (6) Trigger Render Deploy Hooks. The `.env` is always deleted in the `always` post-block.

**Q47. What is the difference between a Docker Image and a Docker Container?**
> An Image is a read-only blueprint (like a class). A Container is a running instance of that image (like an object). `docker build` creates the image; `docker run`/`docker-compose up` starts the container.

**Q48. What does `depends_on` in `docker-compose.yml` do?**
> It controls startup order. `frontend: depends_on: backend` ensures the backend container starts before the frontend container boots, preventing immediate API connection errors at startup.

**Q49. Why is tagging Docker images with `BUILD_NUMBER` better than using `latest`?**
> Using `latest` overwrites the previous image, making rollbacks impossible. Tagging with the Jenkins build number (e.g., `prescripto-backend:42`) creates an immutable artifact for every build, allowing instant rollback to any previous version.

**Q50. What is a Blue-Green Deployment and how does it achieve zero downtime?**
> Two identical environments: Blue (live) and Green (new). Jenkins deploys to Green, runs health checks, then the load balancer switches all traffic from Blue to Green instantly. If Green fails, traffic reverts to Blue with zero user disruption.

**Q51. How does Jenkins securely handle secrets like `RAZORPAY_KEY_SECRET`?**
> Secrets are stored encrypted in Jenkins' built-in Credentials Manager. The `withCredentials` block temporarily loads them as environment variables (`$env:RAZORPAY_KEY_SECRET`) only during the pipeline stage that needs them, and never logs them.

**Q52. Why do you run `docker-compose down --remove-orphans` at the start of each build?**
> To ensure a clean slate. Previous containers may still hold ports 5000, 5173, or 5174 from an older build. Stopping them first prevents "Address already in use" errors that would fail the new build.

---

## ⚡ 9. Scalability & Performance

**Q53. How would you add caching to the `GET /api/user/getDoctors` endpoint?**
> Use Redis with a TTL of 5 minutes. On first request, fetch from MongoDB and store in Redis. Subsequent requests within 5 minutes return the cached result instantly, reducing database load dramatically for a heavily-read endpoint.

**Q54. How would you implement real-time notifications (e.g., "Doctor confirmed your appointment")?**
> Use WebSockets via Socket.io. When a doctor marks an appointment as complete, the backend emits a `appointment:completed` event to the patient's connected socket. The patient sees an instant in-app notification.

**Q55. What is a CDN and which parts of Prescripto benefit from one?**
> A Content Delivery Network caches static assets on servers globally close to users. The React frontends (compiled HTML/CSS/JS bundles) and Cloudinary images benefit most, since they don't change per-request and can be served from a node geographically near the user.

**Q56. How would you prevent a doctor's dashboard from timing out when loading thousands of appointments?**
> Use pagination (limit results to 20 per page), add a database index on `appointments.docId`, and implement cursor-based pagination for deeper pages. For very large datasets, use MongoDB aggregation pipelines with `$facet` for simultaneous count and data queries.

**Q57. What is connection pooling in MongoDB and why does it matter?**
> Creating a new TCP connection to MongoDB for every API request is expensive. Mongoose maintains a pool of persistent connections (default: 5). Incoming requests reuse connections from the pool, significantly reducing latency under concurrent load.

---

## 🚨 10. Real-World Production Scenarios

**Q58. A patient reports they booked an appointment that the doctor never received. How do you debug this?**
> Check the `appointments` collection directly in MongoDB Atlas for the appointment document. Verify the `docId` is correct. Check the backend API logs for errors during the `bookAppointment` controller. Verify the `slots_booked` update on the doctor document was written atomically.

**Q59. The live Render backend is returning 503 errors. What is your incident response process?**
> (1) Check Render dashboard logs for crash reason. (2) Check MongoDB Atlas connection status. (3) If a code bug, `git revert` the last commit and push to trigger Jenkins pipeline redeployment. (4) If persistent, upgrade from free tier to eliminate cold starts and resource limits.

**Q60. How would you design a notification system to remind patients of upcoming appointments via email/SMS?**
> Add a CRON job (e.g., using `node-cron`) that runs every hour, queries for appointments in the next 24 hours where `reminderSent: false`, sends emails via SendGrid or SMS via Twilio, then marks `reminderSent: true`. This is a background worker pattern, separate from the main Express API.

---

<div align="center">

**📚 Prescripto System Design Questions — 60 Questions Across 10 Categories**

*Covers: Architecture · Database · Auth/Security · Booking · Payments · File Uploads · REST API · DevOps · Scalability · Production Incidents*

</div>
