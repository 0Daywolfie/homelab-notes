# Topool Autos - Dealership Website Rebuild

## Overview
Rebuilt topoolautos.com (RSM Topool Autos, a Lagos-based car dealership) from WordPress into a custom React/Vite/Tailwind storefront with a PHP/MySQL backend. Live, fully functional site currently paused pending real vehicle photos from the business owner before public launch.

## Stack
- Frontend: React + TypeScript (.tsx), Vite, Tailwind CSS, react-router
- Backend: PHP, MySQL
- Hosting: Qservers (Nigerian cPanel shared hosting - no Node runtime, no subdomain support on this plan tier)
- Email: PHPMailer over SMTP (cPanel mail() function is disabled on this host, standard PHP mail() would have silently failed)

## Architecture

### Frontend routes
/, /vehicle/:id, /about, /contact, /rental

### Key components
Navbar, Hero, VehicleGrid, VehicleCard, VehicleDetail, EnquiryDrawer (batches multiple vehicle enquiries into a single submission), MobileMenu, About, Contact, Rental, WhatsAppButton, LoadingCar, Footer

### Backend API
- GET /api/vehicles.php - returns id, make, model, year, price, bodyType, images (up to 5 per vehicle, 3 per rental), description
- POST /api/send-enquiry.php - requires name, valid email, non-empty vehicles field; accepts phone, message; includes a honeypot field for basic bot filtering

### Admin
Database-backed admin panel at /admin/login.php with photo upload for inventory management - no manual file uploads or direct DB edits needed for day-to-day vehicle listing changes.

## Hosting constraints and how they were solved
Shared cPanel hosting on Qservers came with two real limitations that shaped the build:

1. No Node runtime, no subdomain support - meant the React app had to be built statically and deployed as static assets rather than run as a live Node server, and the API had to live on PHP under the same domain rather than a separate subdomain/service.

2. PHP mail() function disabled - the standard, simplest way to send email from PHP silently fails on this host. Solved by integrating PHPMailer configured for SMTP instead, so the enquiry form genuinely delivers messages rather than appearing to work while failing invisibly in production.

## Business-facing features
- Naira pricing throughout
- Real contact details: phone, email, Ikoyi Lagos address, embedded Google Maps
- WhatsApp Business integration (direct WhatsApp contact button)
- Custom night drive dark/glass visual theme
- Trademark mark and custom car-themed favicon
- Google Business Profile ownership transferred to the business owner

## Security practices
Real database and SMTP credentials live in server/config.php, which is gitignored and has never been pushed to the public repository. Code is versioned at github.com/0Daywolfie/storefront-placeholder (kept private, under a placeholder repo name rather than the real business name).

## Status
Paused as of September 2026, pending the business owner uploading real vehicle photos via the admin panel. No further code work is required for that step - it is purely a content/data task for the owner. Business hours on the Contact page remain a placeholder pending confirmation from the owner.

## Planned next phase - Showroom Daylight redesign
A lighter, editorial visual direction chosen over the current dark/glass theme, plus a new feature: an interactive monthly-payment financing pre-qualification calculator on vehicle detail pages. Built on a separate branch, kept behind a feature flag until launch-ready, pending the owner's approval before deploying live.

## Future consideration (deliberately deferred)
Chief does affiliate marketing for Expedia and is considering a future airport car rental service for Topool Autos, potentially bundling Expedia hotel/rental affiliate links. Deliberately not added to the site yet - the decision was to wait until that service is real rather than dilute the dealership's trust signal with unrelated affiliate content before it's genuinely relevant.
