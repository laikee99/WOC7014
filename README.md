# DRIS (Disaster Response Information System) Project Documentation

**Student Name:** DU WENTAO  
**Student ID:** 24084310  
**Framework:** Django Web Framework  
**UI Framework:** Semantic UI CSS  
**Database:** SQLite  
**Architecture:** MVC  

## Table of Contents
1. [Project Overview](#project-overview)
2. [Data Model Design](#data-model-design)
3. [System Architecture](#system-architecture)
4. [User Interface Design](#user-interface-design)
5. [Navigation Flow](#navigation-flow)
6. [UI Flow Diagrams](#ui-flow-diagrams)
7. [Implementation Details](#implementation-details)
8. [Security Features](#security-features)
9. [Installation Guide](#installation-guide)
10. [User Roles and Permissions](#user-roles-and-permissions)

## Project Overview

The Disaster Response Information System (DRIS) is a centralized web-based platform developed for the National Disaster Management Agency (NADMA) to improve coordination and communication during disasters in Malaysia. The system facilitates interaction between three main user groups:

- **Citizens**: Report disasters and request aid
- **Volunteers**: Register availability and assist in emergencies
- **Authorities**: Monitor, manage, and coordinate disaster response
- **Admin**: Verify informations


## Q1 Data Model Design

### Entity Relationship Overview

The DRIS data model consists of six main entities with carefully designed relationships to support all functional requirements.

### 1. User (Extended with UserProfile)
- **Fields:**
  - username (CharField) - Unique identifier
  - email (EmailField) - Contact email
  - first_name (CharField) - User's first name
  - last_name (CharField) - User's last name
  - password (CharField) - Hashed password
  - date_joined (DateTimeField) - Registration timestamp

### 2. UserProfile
- **Fields:**
  - user (OneToOneField to User) - Link to auth user
  - role (CharField: citizen/volunteer/authority) - User's role in system
  - skills (TextField) - Comma-separated skills for volunteers
  - availability (BooleanField) - Availability status for volunteers
  - created_at (DateTimeField) - Profile creation time
  - updated_at (DateTimeField) - Last update time
- **Relationships:**
  - One-to-One with User

### 3. DisasterReport
- **Fields:**
  - reporter (ForeignKey to User) - Person reporting disaster
  - disaster_type (CharField: flood/fire/earthquake/landslide/haze/other)
  - severity (CharField: low/medium/high/critical)
  - location (CharField) - Text description of location
  - gps_coordinates (CharField) - Optional GPS coordinates
  - description (TextField) - Detailed description
  - timestamp (DateTimeField) - When disaster occurred
  - is_verified (BooleanField) - Verification status
  - created_at (DateTimeField) - Report creation time
  - updated_at (DateTimeField) - Last update time
- **Relationships:**
  - Many-to-One with User (reporter)

### 4. Shelter
- **Fields:**
  - name (CharField) - Shelter name
  - location (CharField) - Physical address
  - capacity (IntegerField) - Maximum capacity
  - current_occupancy (IntegerField) - Current number of occupants
  - available_spaces (IntegerField - calculated) - Auto-calculated field
  - facilities (TextField) - Available facilities description
  - contact_person (CharField) - Person in charge
  - contact_number (CharField) - Contact phone
  - is_active (BooleanField) - Active status
  - created_at (DateTimeField) - Creation time
  - updated_at (DateTimeField) - Last update time

### 5. AidRequest
- **Fields:**
  - requester (ForeignKey to User) - Person requesting aid
  - request_type (CharField: food/water/shelter/medical/clothing/rescue/other)
  - description (TextField) - Detailed needs description
  - location (CharField) - Where aid is needed
  - urgency_level (CharField: low/medium/high/critical)
  - number_of_people (IntegerField) - People affected
  - status (CharField: pending/assigned/in_progress/completed/cancelled)
  - assigned_shelter (ForeignKey to Shelter, nullable) - If shelter assigned
  - created_at (DateTimeField) - Request creation time
  - updated_at (DateTimeField) - Last update time
- **Relationships:**
  - Many-to-One with User (requester)
  - Many-to-One with Shelter (optional)

### 6. VolunteerAssignment
- **Fields:**
  - volunteer (ForeignKey to User) - Assigned volunteer
  - aid_request (ForeignKey to AidRequest) - Related aid request
  - assigned_by (ForeignKey to User) - Authority who assigned
  - task_description (TextField) - Specific tasks to perform
  - status (CharField: assigned/accepted/in_progress/completed/cancelled)
  - assigned_at (DateTimeField) - Assignment time
  - completed_at (DateTimeField, nullable) - Completion time
- **Relationships:**
  - Many-to-One with User (volunteer)
  - Many-to-One with AidRequest
  - Many-to-One with User (assigned_by)

### ER Diagram
```mermaid

erDiagram
    USER ||--o| USER_PROFILE : has
    USER ||--o{ DISASTER_REPORT : reports
    USER ||--o{ AID_REQUEST : requests
    USER ||--o{ VOLUNTEER_ASSIGNMENT : "assigned as volunteer"
    USER ||--o{ VOLUNTEER_ASSIGNMENT : "assigned by"
    
    AID_REQUEST ||--o{ VOLUNTEER_ASSIGNMENT : "has assignments"
    AID_REQUEST }o--o| SHELTER : "assigned to"
    
    USER {
        int id PK
        string username
        string email
        string first_name
        string last_name
        string password
        datetime date_joined
    }
    
    USER_PROFILE {
        int id PK
        int user_id FK
        string role
        text skills
        boolean availability
        datetime created_at
        datetime updated_at
    }
    
    DISASTER_REPORT {
        int id PK
        int reporter_id FK
        string disaster_type
        string severity
        string location
        string gps_coordinates
        text description
        datetime timestamp
        boolean is_verified
        datetime created_at
        datetime updated_at
    }
    
    SHELTER {
        int id PK
        string name
        string location
        int capacity
        int current_occupancy
        int available_spaces
        text facilities
        string contact_person
        string contact_number
        boolean is_active
        datetime created_at
        datetime updated_at
    }
    
    AID_REQUEST {
        int id PK
        int requester_id FK
        string request_type
        text description
        string location
        string urgency_level
        int number_of_people
        string status
        int assigned_shelter_id FK
        datetime created_at
        datetime updated_at
    }
    
    VOLUNTEER_ASSIGNMENT {
        int id PK
        int volunteer_id FK
        int aid_request_id FK
        int assigned_by_id FK
        text task_description
        string status
        datetime assigned_at
        datetime completed_at
    }
```


## System Architecture

The DRIS follows the Model-View-Controller (MVC) architecture pattern:

### Models
- Define data structure and business logic
- Handle database interactions
- Implement data validation
- Auto-calculate fields (e.g., shelter availability)

### Controllers
- Process HTTP requests
- Implement business logic
- Handle form processing
- Manage user authentication and authorization
- Role-based access control

### Views
- Render dynamic HTML content
- Implement Semantic UI components
- Handle user interface presentation
- Support responsive design

### Key Architectural Decisions
1. **Modular Design**: Separate apps for different functionalities
2. **Reusable Components**: Custom template tags for skills display
3. **Security First**: CSRF protection, role-based views
4. **Scalable Structure**: Easy to add new disaster types or user roles

## Q2 User Interface Design

### Design Principles
- **Responsive Design**: Mobile-friendly using Semantic UI
- **Intuitive Navigation**: Role-based menu system
- **Visual Feedback**: Color-coded severity and status labels
- **Accessibility**: Clear icons and labels

### Base Template Structure
The base template (`base.html`) includes:
- **Emergency Hotline Banner**: Always visible emergency contacts (999, NADMA: 03-8064 2400)
- **Dynamic Navigation**: Menu items change based on user role
- **Message System**: Success/error notifications
- **Footer**: Emergency info and student details

### Key UI Components

1. **Home Page** (`home.html`)
   User Page
   
   ![127 0 0 1_8000_ (3)](https://github.com/user-attachments/assets/496ae686-de53-48a3-9a4c-6754b6069c10)

   Volunteer Page
   
   ![127 0 0 1_8000_ (4)](https://github.com/user-attachments/assets/78e98f9d-0adf-419a-bb75-876dd4068b21)

   Manager Page
   
   ![127 0 0 1_8000_ (5)](https://github.com/user-attachments/assets/1b91bd90-7ed1-40e6-84b6-cd2d237b0a89)


   - Hero section with three action cards
   - Recent verified disaster reports
   - Available emergency shelters
   - Call-to-action for different user types
   - Login/Register
   ![image](https://github.com/user-attachments/assets/5f270202-9b3c-45ad-be20-d954fe11fcac)

   ![image](https://github.com/user-attachments/assets/ebd23319-0917-4f71-bfd9-044e14fcc5f6)



2. **Disaster Reports** (`disaster_reports.html`)
   - Advanced filter form with multiple criteria
  
  ![image](https://github.com/user-attachments/assets/1d717f27-dae3-4b5b-b5f2-bc62144e3728)
  
  ![image](https://github.com/user-attachments/assets/9813e182-1969-4ba9-b4b4-75c0f3204fdf)
  

   - Icon-based disaster type visualization
   - Color-coded severity labels
   - Verification status indicators

3. **Shelters** (`shelters.html`)

   ![image](https://github.com/user-attachments/assets/fe5686c8-7e38-4ee9-871f-9703844f3155)

   - Search functionality by location
   
   ![image](https://github.com/user-attachments/assets/dad11fb5-d6bb-4d2c-b4e4-a5a44339f1bc)

   - Card-based layout for visual appeal
   - Real-time availability display
   
   ![127 0 0 1_8000_shelters_ (1)](https://github.com/user-attachments/assets/c86dda41-1556-4807-9f3e-8b506b4fbfbb)
  

   - Quick contact and directions buttons

4. **Aid Requests** (`aid_requests.html`)

   **Volunteer View**
   Can see aids assigned to this volunteer.

   ![127 0 0 1_8000_aid-requests_9_ (1)](https://github.com/user-attachments/assets/9e1ceb4d-95c9-48e2-bf81-1e63a2b5fbc3)


   **Normal User**
   Can see and edit aids under this user account, can create new one.

   ![image](https://github.com/user-attachments/assets/a30594a1-c76b-4179-9e5b-d2413a43a92e)
   
   ![image](https://github.com/user-attachments/assets/8bec9c0d-d63b-44a9-a426-013194717319)
   
   ![image](https://github.com/user-attachments/assets/cc7edbb6-e131-4dbc-9b59-b6db1e9e55a3)


   ![image](https://github.com/user-attachments/assets/3145011b-e07c-4771-85d1-0bc82720652f)
  
   - Role-based view filtering
   - Table layout for efficient data display
   - Status tracking with color codes
   - Integrated volunteer assignment info

6. **Volunteer Dashboard** (`volunteer_dashboard.html`)


   ![image](https://github.com/user-attachments/assets/10438ae2-6cab-4a5e-83d3-fcb3bea4dadf)

   ![image](https://github.com/user-attachments/assets/6cd4c641-7ed0-44bc-876f-71072f7480f4)

   ![image](https://github.com/user-attachments/assets/e93c1006-42d4-4a6c-a952-e75ab79310b6)

   ![image](https://github.com/user-attachments/assets/f53ad75c-9b20-452e-bd4a-984e95870cda)

   ![image](https://github.com/user-attachments/assets/bb879e2d-aa1e-43d4-8fde-bf7f1842c03a)

   ![image](https://github.com/user-attachments/assets/6252719a-2bd5-4f0e-a63f-d9d559051aa3)


   ![image](https://github.com/user-attachments/assets/2e232dad-43a5-4587-b8c4-1168a82636f6)


   Volunteer manage assignments
   
   ![127 0 0 1_8000_volunteer-assignments_](https://github.com/user-attachments/assets/7847fad8-83a9-4d8e-8e01-bd01b60b525d)


    
   Volunteer/Manager view aid details
   
   ![image](https://github.com/user-attachments/assets/a7dbe55e-7ce7-488e-a9a2-fdac482a4fb1)


   - Statistics overview cards
   - Pending requests management
   - Visual volunteer cards with skill tags
   - Assignment history tracking

UI Flow
```mermaid
flowchart TB
    subgraph "Landing Page UI"
        L1[Emergency Hotline Banner]
        L2[Navigation Menu]
        L3[Hero Section]
        L4[Recent Disasters]
        L5[Available Shelters]
        L6[Call to Action]
    end
    
    subgraph "Disaster Report UI"
        D1[Filter Form]
        D2[Report List]
        D3[Report Card]
        D4[Status Labels]
        D5[Create Report Button]
    end
    
    subgraph "Aid Request UI"
        A1[Request Form]
        A2[Request Table]
        A3[Status Tracking]
        A4[Assignment Info]
    end
    
    L3 --> D1
    L3 --> A1
    
    style L1 fill:#f66,stroke:#333,stroke-width:2px
    style D1 fill:#6cf,stroke:#333,stroke-width:2px
    style A1 fill:#fc6,stroke:#333,stroke-width:2px
```


## Q3 Navigation Flow

### 1. General User Flow
```
Home Page → Login/Register → Role-based Dashboard
```

### 2. Citizen Navigation Flow
- **Main Actions**: Report disasters, request aid, track status
- **Access Points**: Home → Create forms → View own submissions

### 3. Volunteer Navigation Flow
- **Main Actions**: Update profile/skills, toggle availability, view assignments
- **Access Points**: Profile → Skills management → Assignment tracking

### 4. Authority Navigation Flow
- **Main Actions**: Monitor all activities, assign volunteers, manage system
- **Access Points**: Dashboard → Pending requests → Volunteer assignment

## Navigation Diagram

```mermaid
graph TD
    A[Home Page] --> B{User Logged In?}
    B -->|No| C[Public View]
    B -->|Yes| D[Check User Role]
    
    C --> E[View Disasters]
    C --> F[View Shelters]
    C --> G[Login/Register]
    
    D --> H{Role Type}
    H -->|Citizen| I[Citizen Dashboard]
    H -->|Volunteer| J[Volunteer Dashboard]
    H -->|Authority| K[Authority Dashboard]
    
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style I fill:#9f9,stroke:#333,stroke-width:2px
    style J fill:#ff9,stroke:#333,stroke-width:2px
    style K fill:#f99,stroke:#333,stroke-width:2px
```

```mermaid
graph LR
    A[Citizen Functions] --> B[Report Disaster]
    A --> C[Request Aid]
    A --> D[View My Reports]
    A --> E[View My Requests]
    
    B --> B1[Fill Form]
    B1 --> B2[Submit Report]
    B2 --> B3[View Status]
    
    C --> C1[Select Aid Type]
    C1 --> C2[Provide Details]
    C2 --> C3[Submit Request]
    C3 --> C4[Track Status]
    
    style A fill:#9f9,stroke:#333,stroke-width:2px
```


```mermaid

graph LR
    A[Volunteer Functions] --> B[Update Profile]
    A --> C[Set Availability]
    A --> D[View Assignments]
    A --> E[Update Task Status]
    
    B --> B1[Add Skills]
    B1 --> B2[Save Profile]
    
    C --> C1[Toggle Available]
    C1 --> C2[Update Status]
    
    D --> D1[View Details]
    D1 --> D2[Accept/Reject]
    
    style A fill:#ff9,stroke:#333,stroke-width:2px

```


```mermaid

graph LR
    A[Authority Functions] --> B[Volunteer Management]
    A --> C[Aid Request Management]
    A --> D[System Monitoring]
    A --> E[User Management]
    
    B --> B1[View Available Volunteers]
    B1 --> B2[Assign to Requests]
    B2 --> B3[Monitor Progress]
    
    C --> C1[View Pending Requests]
    C1 --> C2[Assign Volunteers]
    C2 --> C3[Update Status]
    
    style A fill:#f99,stroke:#333,stroke-width:2px

```

### Landing Page Structure
1. **Emergency Banner** (Red) - Always visible
2. **Navigation Bar** - Role-adaptive menu
3. **Hero Section** - Three main action cards
4. **Content Sections** - Recent disasters & shelters
5. **Footer** - Emergency contacts & info

### Form Flow Pattern
1. **Entry Point** - Button/link from navigation
2. **Form Display** - Semantic UI styled forms
3. **Validation** - Client and server-side
4. **Submission** - POST with CSRF token
5. **Feedback** - Success/error messages
6. **Redirect** - To relevant listing page

### Data Display Pattern
1. **Filter/Search** - Top of page controls
2. **Results Display** - Cards or tables
3. **Status Indicators** - Color-coded labels
4. **Action Buttons** - Context-sensitive
5. **Details** - Expandable or modal views

## Implementation Details

### Key Features Implemented

1. **User Management**
   - Django's built-in authentication extended with UserProfile
   - Role-based registration with automatic profile creation
   - Secure login/logout with session management
   - Profile management with role-specific fields

2. **Disaster Reporting**
   - Comprehensive form with all disaster types
   - GPS coordinate capture support
   - Severity classification system
   - Admin verification workflow

3. **Aid Request Management**
   - Multi-type request system
   - Urgency level prioritization
   - Status tracking through lifecycle
   - Volunteer assignment integration

4. **Volunteer Coordination**
   - Skill tags system (comma-separated)
   - Visual skill display with labels
   - Availability toggle feature
   - Assignment tracking with status updates

5. **Shelter Directory**
   - Real-time availability calculation
   - Search by location functionality
   - Contact integration
   - Google Maps directions link

6. **Administrative Dashboard**
   - Comprehensive volunteer overview
   - Pending request management
   - Visual assignment workflow
   - Statistics display

### Technical Implementation

1. **Custom Template Tags**
   - `split` filter for skill parsing
   - `trim` filter for cleanup
   - Located in `templatetags/custom_filters.py`

2. **Form Classes**
   - `UserRegistrationForm` - Extended user creation
   - `DisasterReportForm` - Report submission
   - `AidRequestForm` - Aid request creation
   - `VolunteerProfileForm` - Profile updates
   - Filter forms for search functionality

3. **View Functions**
   - Role-based access decorators
   - Context-aware data filtering
   - Form processing with validation
   - Success/error message handling

## Security Features

1. **Authentication**
   - Django's robust authentication system
   - Password strength validation
   - Session-based authentication
   - Secure password hashing

2. **Authorization**
   - Role-based access control at view level
   - User profile role checking
   - Restricted actions by role
   - Admin-only functions protected

3. **Data Protection**
   - CSRF token on all forms
   - SQL injection prevention via ORM
   - XSS protection in templates
   - Secure default settings

4. **Best Practices**
   - Login required decorators
   - Permission checking in views
   - Secure form handling
   - Input validation
  
5. **Role Based Control**
   
   - Frontend Role
     
     <img width="1503" alt="image" src="https://github.com/user-attachments/assets/8efb3908-5c9e-4acf-ab18-17f88da6658a" />
     <img width="758" alt="image" src="https://github.com/user-attachments/assets/c26423a9-64c8-4e8a-94bf-09506bf56cd4" />

   - Group Role
     
     <img width="765" alt="image" src="https://github.com/user-attachments/assets/f083b4f6-c545-4e99-9429-0875aff470e1" />
     <img width="1497" alt="image" src="https://github.com/user-attachments/assets/4c5e999f-c141-4de1-9e7f-9b108d4e877b" />


## Installation Guide

### Prerequisites
- Python 3.8 or higher
- pip package manager
- Virtual environment tool

## User Roles and Permissions

### Citizen Role
**Permissions:**
- Create disaster reports
- Submit aid requests
- View own submissions
- Update own profile
- View all public information

**Restrictions:**
- Cannot assign volunteers
- Cannot verify reports
- Cannot access admin functions

### Volunteer Role
**Permissions:**
- All citizen permissions
- Update skills and availability
- View assigned tasks
- Update task status
- Access volunteer resources

**Restrictions:**
- Cannot assign other volunteers
- Cannot modify aid requests
- Cannot access authority functions

### Authority Role
**Permissions:**
- All system access
- Assign volunteers to requests
- Verify disaster reports
- Manage all users via admin
- View system statistics
- Update any record status

**Special Features:**
- Volunteer dashboard access
- Assignment management
- System monitoring tools

## Modular Architecture

The application follows Django's modular design principles:

1. **App Structure**
   - `disaster_response` app contains all functionality
   - Clear separation of concerns
   - Reusable components

2. **Template Inheritance**
   - Base template for consistency
   - Block-based content injection
   - Shared navigation and footer

3. **Static Files**
   - Organized CSS/JS structure
   - CDN for external libraries
   - Custom styles in base template

4. **Database Design**
   - Normalized structure
   - Efficient relationships
   - Calculated fields where appropriate

## Future Enhancements

1. **Technical Improvements**
   - REST API for mobile apps
   - Real-time notifications
   - WebSocket for live updates
   - Map integration for visualization

2. **Feature Additions**
   - SMS/Email alerts
   - Resource inventory tracking
   - Multi-language support
   - Advanced analytics dashboard

3. **Scalability**
   - Database optimization
   - Caching implementation
   - Load balancing ready
   - Microservices architecture

---

**Note:** 

admin/admin
user/RnEqeCgb7d6x295
volunteer/4B-2Qr7amx-MPsv
manager/qdXEsU9N?FDMp2i

Dashboard: /admin
