# 📝 Patient Registration Flow

This document explains the complete, end-to-end process of what happens when a new patient creates an account on the **Prescripto** platform.

## 🔄 The Complete Procedure

### 1. Frontend: User Input
**File:** `frontend/src/pages/Login.jsx`
- The user navigates to the login page and selects the **"Sign Up"** option.
- They fill out a form with their **Full Name**, **Email**, and **Password**.
- Clicking "Create Account" triggers the `onSubmitHandler` function.
- The default browser form submission is prevented (`event.preventDefault()`).

### 2. Frontend: API Request
**File:** `frontend/src/pages/Login.jsx`
- The frontend sends an HTTP `POST` request using `axios` to the backend endpoint `[backendUrl]/api/user/register`.
- The request body contains the user's data: `{ name, email, password }`.

### 3. Backend: Route & Controller
**Route File:** `backend/routes/userRoute.js` (Maps `/register` to the controller)
**Controller File:** `backend/controllers/userController.js` (Function: `registerUser`)
- The request is routed to the `registerUser` function.
- **Validation Steps:**
  - **Missing Data:** Checks if `name`, `email`, and `password` are provided.
  - **Email Format:** Validates the email using the `validator` package (`validator.isEmail()`).
  - **Password Strength:** Ensures the password is at least 8 characters long.
- If any validation fails, the backend immediately returns a response like `{ success: false, message: "Error message" }`.

### 4. Backend: Password Security
**File:** `backend/controllers/userController.js`
- The backend generates a secure salt with a cost factor of 10 (`bcrypt.genSalt(10)`).
- The plain-text password is mathematically hashed using this salt (`bcrypt.hash(password, salt)`).

### 5. Backend: Database Storage
**Model File:** `backend/models/userModel.js`
**Controller File:** `backend/controllers/userController.js`
- A new user object is constructed with the `name`, `email`, and the **hashed password**.
- This object is saved as a new document in the MongoDB Atlas database (`newUser.save()`).

### 6. Backend: Authentication Token
**File:** `backend/controllers/userController.js`
- A JSON Web Token (JWT) is generated using `jwt.sign()`.
- The payload of the token contains the newly created user's unique database ID (`{ id: user._id }`).
- The token is signed securely using the `JWT_SECRET` environment variable.
- The backend sends a successful response back to the frontend: `{ success: true, token: "..." }`.

### 7. Frontend: Completing the Flow
**Component File:** `frontend/src/pages/Login.jsx`
**Context File:** `frontend/src/context/AppContext.jsx`
- The frontend receives the response in the `onSubmitHandler`.
- If `success` is true, the `token` is saved to the browser's `localStorage` to keep the user logged in across page reloads.
- The application's global state is updated with the new token (`setToken(data.token)`).
- A `useEffect` hook detects that the token is now set and automatically redirects (`navigate('/')`) the user to the home page.
- If there was an error during the process, a popup notification (`toast.error()`) displays the error message to the user.

---

## 📊 Process Flowchart

Below is a detailed Mermaid flowchart visualizing the step-by-step registration process. It highlights the Frontend, Backend, and Database layers.

```mermaid
graph TD
    %% Define Styles
    classDef frontend fill:#E3F2FD,stroke:#1565C0,stroke-width:2px,color:#000
    classDef backend fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#000
    classDef database fill:#FFF3E0,stroke:#E65100,stroke-width:2px,color:#000
    classDef error fill:#FFEBEE,stroke:#C62828,stroke-width:2px,color:#000
    
    %% User Actions
    A([User navigates to Sign Up]) --> B[/Enters Name, Email, Password/]
    B --> C{Clicks 'Create Account'}
    
    %% Frontend execution
    C -->|Submits Form| D[Frontend: Prevent Default Submit]:::frontend
    D --> E[Frontend: POST /api/user/register]:::frontend
    
    %% Backend Validation
    E --> F{Backend: Are fields missing?}:::backend
    
    F -->|Yes| ERR1[Return Error: 'Missing Details']:::error
    F -->|No| G{Backend: Is Email Valid?}:::backend
    
    G -->|No| ERR2[Return Error: 'Enter a valid email']:::error
    G -->|Yes| H{Backend: Password >= 8 chars?}:::backend
    
    H -->|No| ERR3[Return Error: 'Enter a strong password']:::error
    H -->|Yes| I[Backend: Generate Salt & Hash Password]:::backend
    
    %% Database Ops
    I --> J[(MongoDB: Save New User)]:::database
    
    %% Token Generation & Completion
    J --> K[Backend: Generate JWT Token]:::backend
    K --> L[Backend: Return success: true, token]:::backend
    
    %% Frontend Response Handling
    ERR1 --> ERR_UI(Frontend: Show Error Toast):::error
    ERR2 --> ERR_UI
    ERR3 --> ERR_UI
    
    L --> M[Frontend: Save token to localStorage]:::frontend
    M --> N[Frontend: Update AppContext]:::frontend
    N --> O([Frontend: Redirect to Home Page /]):::frontend
```

---

## 🔄 Sequence Diagram

Below is a detailed Mermaid sequence diagram visualizing the registration flow across the different layers of the application over time.

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Frontend as React App (Login.jsx)
    participant Backend as Express API (userController)
    participant DB as MongoDB Atlas

    User->>Frontend: Enters Name, Email, Password
    User->>Frontend: Clicks "Create Account"
    Frontend->>Backend: POST /api/user/register {name, email, password}
    
    activate Backend
    Backend->>Backend: Check for missing fields
    Backend->>Backend: Validate Email Format (validator)
    Backend->>Backend: Validate Password Length (>= 8)
    
    alt Validation Fails
        Backend-->>Frontend: {success: false, message: "Error details"}
        Frontend-->>User: Show Error Toast
    else Validation Passes
        Backend->>Backend: Generate Salt (bcrypt)
        Backend->>Backend: Hash Password (bcrypt)
        Backend->>DB: Save New User {name, email, hashedPassword}
        activate DB
        DB-->>Backend: Return User Object (with _id)
        deactivate DB
        Backend->>Backend: Generate JWT token with _id
        Backend-->>Frontend: {success: true, token: "jwt_token"}
    end
    deactivate Backend

    alt Response is Successful
        Frontend->>Frontend: Save token to localStorage
        Frontend->>Frontend: Update Context (setToken)
        Frontend-->>User: Redirect to Home Page (/)
    end
```
