# a Proposal of a modular system.

## Base System (Foundation).

### Feature 1 — Landing Page
| ID | Use Case | User Story | Acceptance Criteria |
|----|----------|------------|---------------------|
| UC1.1 | Platform Overview | As a potential user, I want to view a landing page with platform information so that I can learn about the app and decide to sign up. | Given a visitor accesses the landing page<br>When they browse sections (hero, features, CTA)<br>Then feature overviews are displayed, and "Sign Up" button redirects to signup screen.<br>Edge: Mobile view → responsive stacking of cards.<br>Error: Page load failure → show fallback static text with retry button. |

### Feature 2 — Authentication
| ID | Use Case | User Story | Acceptance Criteria |
|----|----------|------------|---------------------|
| UC2.1 | Signup Basic Info | As a new user, I want to enter basic details (email, phone, password, license/document) so that I can start creating an account. | Given a new user on signup step 1<br>When they submit valid details and upload document<br>Then form validates in real-time, and proceeds to OTP verification.<br>Edge: Missing fields → disabled "Next" button with highlighted errors.<br>Error: Duplicate email/phone → inline error "Already registered? Log in" with link. |
| UC2.2 | OTP Verification | As a new user, I want to verify email or phone via OTP so that my contact details are confirmed securely. | Given OTP sent during signup<br>When user enters correct 6-digit OTP (tabs for email/phone)<br>Then verification succeeds, proceeds to profile completion.<br>Edge: Resend OTP → new code sent after 60s cooldown.<br>Error: Invalid OTP (3 attempts) → lockout modal "Too many attempts. Wait 5 mins." with shake animation. |
| UC2.3 | Profile Completion | As a new user, I want to complete my profile (name, business type, address, avatar) so that my account is fully set up pending approval. | Given OTP verified<br>When user submits valid profile details<br>Then account is created, success modal shown, and admin notified for approval.<br>Edge: Optional fields skipped → no error, but reminder toast.<br>Error: Invalid data (e.g., short name) → error list at top, scroll to first issue. |
| UC2.4 | Post-Signup Confirmation | As a new user, I want a confirmation after signup so that I know the status and next steps. | Given profile submitted<br>When modal/page loads<br>Then displays "Account pending approval" with links to login/home.<br>Edge: Email notification sent → confirm in logs (system).<br>Error: None (static page). |
| UC2.5 | Login | As an existing user, I want to log in with email/phone and password so that I can access my dashboard securely. | Given a registered user on login screen<br>When correct credentials entered<br>Then user is authenticated and redirected to role-based dashboard.<br>Edge: "Forgot Password?" link → opens recovery modal.<br>Error: Invalid credentials (3 attempts) → show CAPTCHA and temporary lockout. |
| UC2.6 | Password Recovery | As an existing user, I want to recover my password via OTP so that I can regain access if forgotten. | Given "Forgot Password" clicked<br>When email/phone submitted, OTP verified, new password set<br>Then password is reset, success message shown, and redirect to login.<br>Edge: Step indicators for multi-step process.<br>Error: Invalid email/phone → "Not found" error; OTP fail → resend option. |

### Feature 3 — Authorization (RBAC)
| ID | Use Case | User Story | Acceptance Criteria |
|----|----------|------------|---------------------|
| UC3.1 | Role-Based Dashboard Routing | As a user, I want automatic routing to my role-specific dashboard upon login so that I see relevant content immediately. | Given user logs in with a role<br>When URL/path and role checked<br>Then redirected to appropriate dashboard (e.g., user vs. admin).<br>Edge: Multi-role user → header dropdown to switch roles, confirms and reloads.<br>Error: Invalid/unauthorized role → 403 error page "Contact Admin" with logout. |
| UC3.2 | Define Roles | As an Admin, I want to create, edit, or delete roles with permissions so that access control reflects business needs. | Given admin on "Define Roles" tab<br>When they create/edit a role (name, description, inheritance, permissions matrix)<br>Then role is persisted, list refreshes, and success toast shown.<br>Edge: Inheritance selected → pre-fills permissions, overridable.<br>Error: Duplicate name → inline error "Name in use"; delete in use → modal "Unassign from users first." |
| UC3.3 | Permission Granularity | As an Admin, I want to set fine-grained permissions (entity/action level) so that least privilege is enforced. | Given role creation/edit modal open<br>When permissions toggled (e.g., Read/Write for Analytics)<br>Then matrix updates, and on save, UI/API blocks unauthorized actions (403).<br>Edge: Constraint checkbox (e.g., one role per system) → adds scoping dropdown.<br>Error: Conflict with inheritance → warning banner "Overriding inherited permissions?" |
| UC3.4 | Assign Roles | As an Admin, I want to assign roles to users or teams with scoping so that access is limited appropriately. | Given admin on "Assign Roles" tab<br>When users selected, role/scope chosen, and "Assign" clicked<br>Then assignments update, current roles column refreshes, toast "Assigned to X users."<br>Edge: Bulk select/unassign → confirm modal for changes.<br>Error: Constraint violation (e.g., one role per system) → toast "User already has role in this system - unassign first." |
| UC3.5 | Audit Role Changes | As a system, I want to log role/permission changes and assignments so that admins can review security events. | Given RBAC event occurs (create, assign, delete)<br>When event triggers<br>Then log entry stored with actor, timestamp, before/after state; visible in system analytics.<br>Edge: Export logs → CSV download.<br>Error: Log failure → alert admin via email (if notifications enabled). |

### Feature 4 — Analytics
| ID | Use Case | User Story | Acceptance Criteria |
|----|----------|------------|---------------------|
| UC4.1 | User Analytics Dashboard | As a non-admin user, I want to view basic KPIs (e.g., balance, activity) so that I can track my performance. | Given user on analytics section<br>When data loads<br>Then KPI cards/charts shown (e.g., balance chart, activity list) with refresh/export buttons.<br>Edge: No data → guidance message "Add payments to see stats."<br>Error: Load failure → retry button with error toast. |
| UC4.2 | Admin System Analytics | As an Admin, I want full oversight analytics (multi-user KPIs, logs, alerts) so that I can monitor and resolve system issues. | Given admin on system analytics screen<br>When viewing data (charts, logs table)<br>Then all-user KPIs displayed, filters (user/role/date) applied, resolve button for errors.<br>Edge: Click log → details modal.<br>Error: Unauthorized access → 403 page. |

### Feature 5 — Basic Payment & Payout
| ID | Use Case | User Story | Acceptance Criteria |
|----|----------|------------|---------------------|
| UC5.1 | Balance Check & Top-Up | As a user, I want to view and top up my balance via Moyasar so that I can maintain funds for operations. | Given user on payments section<br>When "Top Up" clicked, amount entered, Moyasar form submitted<br>Then payment processed, balance updated, success toast shown.<br>Edge: Low balance → warning banner "Top up to continue."<br>Error: Transaction rejected → error "Retry?" with log. |
| UC5.2 | Payout Handling | As a user (with permission), I want to initiate payouts via Moyasar so that I can withdraw funds on schedule. | Given payout cycle (e.g., manual trigger if allowed)<br>When amount/bank details entered in modal<br>Then payout scheduled/processed, logs updated.<br>Edge: Cycle info shown (e.g., next payout date).<br>Error: Insufficient balance → disabled button with tooltip; API fail → retry queue. |

### Feature 6 — Profile & Settings (Shared)
| ID | Use Case | User Story | Acceptance Criteria |
|----|----------|------------|---------------------|
| UC6.1 | Profile Management | As a user, I want to view and edit my profile (name, email, phone, avatar, password) so that my details stay current. | Given user on profile screen<br>When edits saved<br>Then changes persisted, confirmation modal shown.<br>Edge: Pencil icons for edit mode.<br>Error: Invalid input → inline validation errors. |

### Feature 7 — Admin-Specific (User Approval & Oversight)
| ID | Use Case | User Story | Acceptance Criteria |
|----|----------|------------|---------------------|
| UC7.1 | User Approval Queue | As an Admin, I want to review and approve/reject new users so that only valid accounts are activated. | Given admin on approval queue screen<br>When approve/reject clicked (view documents in modal)<br>Then status updated, email sent to user.<br>Edge: No pendings → empty state message.<br>Error: Invalid document → reject with reason modal. |
| UC7.2 | Admin Dashboard Access | As an Admin, I want an enhanced dashboard with admin tools so that I can oversee the system. | Given admin login<br>When dashboard loads<br>Then shows admin sidebar (e.g., User Management, Role Editor), overview KPIs.<br>Edge: Search bar for quick navigation.<br>Error: Non-admin access → hidden/403. |

### Feature 8 — General System Behaviors
| ID | Use Case | User Story | Acceptance Criteria |
|----|----------|------------|---------------------|
| UC8.1 | Logout | As an authenticated user, I want to log out securely so that my session ends. | Given any authenticated screen<br>When logout clicked from profile icon<br>Then confirm modal shown, on yes redirects to landing page.<br>Edge: None.<br>Error: Session error → auto-logout with message. |
| UC8.2 | Error Pages | As a user, I want friendly error pages (404, 500, 403) so that issues are handled gracefully. | Given invalid access/load<br>When error occurs<br>Then shows appropriate page (e.g., 404: "Page not found" with home link).<br>Edge: None.<br>Error: None (fallback UI).

### Wireframes of the base system

<img width="2560" height="2752" alt="screen" src="https://github.com/user-attachments/assets/d46b79fe-e41f-4e26-b487-2143f208e8fc" />

<img width="2560" height="1600" alt="screen" src="https://github.com/user-attachments/assets/ea7900b7-40ef-42a7-bab7-c4b28ae2d97e" />
<img width="2560" height="1600" alt="screen" src="https://github.com/user-attachments/assets/2cbeb1d4-caaf-46f8-9eda-683f50020014" />

<img width="2560" height="1600" alt="screen" src="https://github.com/user-attachments/assets/8f26045c-2da4-4b4c-a264-006794b21391" />
<img width="2560" height="1600" alt="screen" src="https://github.com/user-attachments/assets/1a6ae1b0-4853-4ac1-8a47-a3fc1c36d2d4" />
<img width="2560" height="2378" alt="screen" src="https://github.com/user-attachments/assets/ecd304a7-4b8d-4435-adb5-d86738d7b5e8" />
<img width="2560" height="1600" alt="screen" src="https://github.com/user-attachments/assets/b99e0ed7-cfad-4113-a676-424a72a35838" />
<img width="2560" height="1714" alt="screen" src="https://github.com/user-attachments/assets/928b308f-b6a0-43df-add5-e852efadfd17" />
<img width="2560" height="2174" alt="screen" src="https://github.com/user-attachments/assets/513dd022-3f50-4da8-a862-2702bdd3308c" />
<img width="2560" height="1600" alt="screen" src="https://github.com/user-attachments/assets/7b5c27ba-20a3-4399-9560-b2649af69f60" />
<img width="2560" height="2084" alt="screen" src="https://github.com/user-attachments/assets/38a86e93-498d-4366-8300-2579176ffcee" />
<img width="2560" height="2214" alt="screen" src="https://github.com/user-attachments/assets/4622eaf4-9c03-43ce-ae06-d7a4d6d7970d" />

## Plugin 1 — Store & Product Management



| ID | Use Case | User Story | Acceptance Criteria |
|----|----------|------------|---------------------|
| UC1.1 | Store Registration Form | As a Store Owner, I want to enter store details (name, food type, address, workers, license) so that my store can be onboarded and approved. | Given Store Owner on registration screen<br>When valid details submitted (e.g., name, dropdown for food type, address textarea, workers number input, license upload)<br>Then form validates, store profile saved pending approval, admin notified.<br>Edge: Optional workers field skipped → no error, defaults to 0.<br>Error: Invalid data (e.g., missing name) → inline errors, disabled submit; duplicate store name → "Store name already exists" toast. |
| UC1.2 | Store Approval Workflow | As an Admin, I want to review and approve/reject store registrations so that only compliant stores are activated. | Given pending store in approval queue<br>When admin views details (e.g., click row for modal with license preview)<br>Then approve activates store, sends email to owner; reject prompts reason modal, notifies owner.<br>Edge: No pendings → empty state "No stores awaiting approval."<br>Error: Invalid license (e.g., poor quality) → flag in modal, re-upload request sent to owner. |
| UC1.3 | Product Create/Upload Form | As a Store Owner, I want to create/upload product details (basic info, images, stock, SKU) so that products can be submitted for approval and inventory management. | Given Store Owner on product create screen (post-store approval)<br>When details entered (name/text, images drag-drop/multiple, stock number, SKU text with uniqueness check)<br>Then validates (e.g., image size <5MB), saves pending approval, syncs to inventory queue.<br>Edge: Multiple images → thumbnail previews with reorder/delete.<br>Error: Bad data (e.g., negative stock) → validation error; upload fail → "File type not supported (JPEG/PNG only)" message. |
| UC1.4 | Product Approval Workflow | As an Admin, I want to review and approve/reject new products so that only valid products are listed in inventory and marketplace. | Given pending product in queue<br>When admin reviews (modal with details, images carousel, stock/SKU info)<br>Then approve adds to live inventory, notifies owner; reject with reason, allows resubmit.<br>Edge: Bulk approve/reject → checkboxes for multiple selections.<br>Error: Duplicate SKU → auto-reject with "Duplicate detected" log. |
| UC1.5 | Product List & Read/View | As a Store Owner, I want to list and view my products with search/filter/sort/export so that I can manage large inventories efficiently. | Given Store Owner on products list screen<br>When searching (e.g., by name/SKU), filtering (e.g., by food type/low stock), sorting (e.g., by stock DESC)<br>Then displays paginated table (name, SKU, stock gauge, images thumbnail, actions), export button (CSV/PDF).<br>Edge: No products → empty state "No products yet - create one."<br>Error: Search fail (e.g., network) → retry button with warning. |
| UC1.6 | Product Update/Edit | As a Store Owner, I want to edit existing product details (info, images, stock) so that I can update inventory without re-uploading. | Given Store Owner selects product from list<br>When edits in modal/form (update name, add/remove images, adjust stock)<br>Then saves changes, triggers re-approval if major (e.g., name change), syncs to inventory.<br>Edge: Minor edits (e.g., stock only) → no approval needed.<br>Error: Invalid update (e.g., duplicate SKU) → inline error; concurrent edit → version conflict modal "Refresh and try again." |
| UC1.7 | Product Delete | As a Store Owner, I want to delete products (single or bulk) so that obsolete items are removed from inventory. | Given Store Owner on list screen<br>When selects product(s), clicks delete (confirm modal)<br>Then removes from DB, notifies warehouse if stocked, logs action.<br>Edge: Bulk delete → checkboxes, confirm with count "Delete X products?"<br>Error: Product in active order → block with error "Cannot delete - in use." |
| UC1.8 | Bulk Product Upload/Import | As a Store Owner, I want to bulk upload products via CSV so that I can add multiple items efficiently for enterprise-scale operations. | Given Store Owner on bulk upload screen<br>When uploads CSV (template download link, columns: name, SKU, stock, food type, image URLs)<br>Then parses, validates rows, submits for batch approval, shows progress bar/report (e.g., "X succeeded, Y failed").<br>Edge: Partial success → list errors per row (e.g., "Row 5: Invalid stock").<br>Error: Invalid CSV format → error toast "Malformed file - download template." |
| UC1.9 | Inventory Views & Updates | As a Store Owner, I want to view inventory updates, stock levels, and KPIs so that I can track product status without manual checks. | Given Store Owner on dashboard/inventory section<br>When data loads (real-time sync from warehouse)<br>Then displays table/cards (product name, SKU, stock level, updates list with timestamps), KPIs (e.g., low stock count, sales velocity chart).<br>Edge: Filter by status (e.g., pending approval) → dynamic table update.<br>Error: Sync failure → retry button, fallback to last known data with warning banner. |
| UC1.10 | System Inventory Sync & Rules | As a system, I want to sync product CRUD actions to live inventory with rules (e.g., SKU mapping, low stock thresholds) so that warehouse and marketplace are updated automatically. | Given product CRUD event (create/update/delete/approve)<br>When sync triggers (background job)<br>Then updates inventory DB, applies rules (e.g., map SKU, flag low stock), notifies supervisor for shelf assignment, logs event.<br>Edge: Manual trigger by admin → override button in queue.<br>Error: Sync conflict (e.g., stock mismatch) → admin alert, rollback with error log. |

## Wireframes of Marketplace

<img width="2560" height="2612" alt="screen" src="https://github.com/user-attachments/assets/6bc63de2-d595-4592-82e7-86cd89d8fcea" />
<img width="2644" height="1600" alt="screen" src="https://github.com/user-attachments/assets/48f14b32-8d35-448a-ba6a-17072d4795ac" />

<img width="2560" height="2632" alt="screen" src="https://github.com/user-attachments/assets/e091c622-8412-4191-9b62-64718fca4c50" />
<img width="2560" height="1754" alt="screen" src="https://github.com/user-attachments/assets/df08b3c8-1a9c-4e1a-aab7-5a0a25b6284a" />
<img width="2560" height="1600" alt="screen" src="https://github.com/user-attachments/assets/c4cb594f-29d5-43dd-823d-064d3da37896" />

