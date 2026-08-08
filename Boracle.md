# Boracle Marketplace

> **Project Type:** Web Application / Marketplace Platform  
> **Tech Stack:** React, Firebase Firestore, JavaScript, CSS/HTML  
> **Role:** Developer  

---

##  Overview
**Boracle Marketplace** was built as an early web marketplace application designed to facilitate peer-to-peer listings, browsing, and item discovery. Developed as part of early project work at California State University, Northridge (CSUN ARCS), this project served as a foundational exploration into front-end component architecture and NoSQL document database integration.
<img width="1424" height="702" alt="322172300-313abb1c-463e-48d4-ae06-ba919d74319e" src="https://github.com/user-attachments/assets/169afec8-61cd-4a7c-9c41-58af9fe8a9e5" />


https://github.com/user-attachments/assets/534c2e58-91e9-48b0-b5e9-65453f047c14


---

##  Tech Stack & Tools

* **Frontend:** React (JSX, Component Architecture)
* **Backend & Database:** Firebase / Firestore (Document Database, Realtime Data Sync)
* **State & Data Management:** React Hooks / Props, Firestore Queries
* **Version Control:** Git & GitHub

---

##  Key Features

* **User Item Browsing:** Dynamic rendered views of available marketplace listings.
* **Firestore Integration:** Real-time data fetching and synchronization for posts/items.
* **Listing Creation:** Form interfaces allowing users to submit new items or requests to the database.
* **Component-Based UI:** Modular React components for structured user experience.
* **Admin Monitoring Tools:** Built-in tracking interfaces for monitoring active database reads and writes.
---

##  Challenges & Architectural Lessons Learned

Looking back at this project, it provided critical early experience in full-stack JavaScript development. Navigating the pitfalls of early decisions provided several key insights:

### 1. NoSQL Schema Design & Firestore Usage
* **The Challenge:** Transitioning to a NoSQL document database like Firestore required thinking differently about data modeling compared to relational databases.
* **Lesson Learned:** Early iterations led to redundant reads and suboptimal query structures. I learned the importance of designing document structures around specific query patterns and minimizing unnecessary database reads.

### 2. State Management & Component Hierarchy
* **The Challenge:** Managing application state across nested React components led to prop-drilling and unnecessary re-renders.
* **Lesson Learned:** This project highlighted the necessity of lifting state efficiently or using dedicated state management patterns (like Context API or Redux) as applications scale in complexity.

### 3. Asynchronous Data Handling & UI Feedback
* **The Challenge:** Coordinating asynchronous Firestore calls with React component lifecycles occasionally caused race conditions or unhandled loading states.
* **Lesson Learned:** Gained a deep understanding of handling promises, managing component loading/error states, and properly cleaning up side effects inside `useEffect`.
### 4. Database Access & Monitoring Tools
* **The Challenge:** Administrative notifications and operational alerts were configured for a previous administrator, leaving gaps in active usage visibility.
* **Lesson Learned:** Built admin-facing control monitors to track database read/write volume in real time, ensuring better visibility into database load and resource consumption.
---

##  What I Would Do Differently Today

If I were to rebuild **Boracle Marketplace** today, I would apply modern design patterns and tooling:
* **TypeScript:** Introduce static typing to enforce clear interfaces for marketplace items and user payloads.
* **Centralized State / Custom Hooks:** Abstract Firestore API interactions into reusable custom React hooks (e.g., `useListings()`) to separate business logic from UI components.
* **Security Rules & Validation:** Implement strict Firestore Security Rules and schema validation (e.g., Zod) on all incoming client payloads.
* **Modern Styling Framework:** Utilize Tailwind CSS or CSS Modules to create a cohesive, scalable design system.

---

##  Repository
* **GitHub:** [CSU-Northridge-ARCS-Dev/boracle-marketplace](https://github.com/CSU-Northridge-ARCS-Dev/boracle-marketplace)
