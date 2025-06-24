# DRIS (Disaster Response Information System) Project Documentation

**Student Name:** DU WENTAO  
**Student ID:** 24084310  
**Framework:** Django Web Framework  
**UI Framework:** Semantic UI CSS  
**Database:** SQLite  
**Architecture:** Model-View-Template (MVT)  

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

### Key Features
- Real-time disaster reporting with GPS location support
- Aid request management system
- Volunteer skill registration and assignment
- Emergency shelter directory with availability tracking
- Role-based access control with secure authentication
- Responsive UI design using Semantic UI CSS

## Data Model Design

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

The DRIS follows the Model-View-Template (MVT) architecture pattern:

### Models
- Define data structure and business logic
- Handle database interactions
- Implement data validation
- Auto-calculate fields (e.g., shelter availability)

### Views
- Process HTTP requests
- Implement business logic
- Handle form processing
- Manage user authentication and authorization
- Role-based access control

### Templates
- Render dynamic HTML content
- Implement Semantic UI components
- Handle user interface presentation
- Support responsive design

### Key Architectural Decisions
1. **Modular Design**: Separate apps for different functionalities
2. **Reusable Components**: Custom template tags for skills display
3. **Security First**: CSRF protection, role-based views
4. **Scalable Structure**: Easy to add new disaster types or user roles

## User Interface Design

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
   - Hero section with three action cards
   - Recent verified disaster reports
   - Available emergency shelters
   - Call-to-action for different user types

2. **Disaster Reports** (`disaster_reports.html`)
   - Advanced filter form with multiple criteria
   - Icon-based disaster type visualization
   - Color-coded severity labels
   - Verification status indicators

3. **Shelters** (`shelters.html`)
   - Search functionality by location
   - Card-based layout for visual appeal
   - Real-time availability display
   - Quick contact and directions buttons

4. **Aid Requests** (`aid_requests.html`)
   - Role-based view filtering
   - Table layout for efficient data display
   - Status tracking with color codes
   - Integrated volunteer assignment info

5. **Volunteer Dashboard** (`volunteer_dashboard.html`)
   - Statistics overview cards
   - Pending requests management
   - Visual volunteer cards with skill tags
   - Assignment history tracking

## Navigation Flow

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

## Installation Guide

### Prerequisites
- Python 3.8 or higher
- pip package manager
- Virtual environment tool
- Git for version control

### Setup Steps

1. **Clone the repository**
   ```bash
   git clone [repository-url]
   cd DRIS_Project
   ```

2. **Use virtual environment**
   ```bash
   python3 -m venv venv
   source venv/bin/activate 
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Run migrations**
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

5. **Create superuser**
   ```bash
   python manage.py createsuperuser
   # Follow prompts to create admin account
   ```

6. **Run development server**
   ```bash
   python manage.py runserver
   # Access at http://localhost:8000
   ```

7. **Access points**
   - Main site: http://localhost:8000
   - Admin panel: http://localhost:8000/admin
   - API endpoints: Not implemented (future enhancement)

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

**Note:** This system is designed for educational purposes demonstrating Django web framework capabilities. Production deployment would require additional security hardening, performance optimization, and compliance with data protection regulations. 
