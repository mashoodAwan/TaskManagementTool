# DuoTasks — Product Requirements Document (PRD)

**Version:** 1.0  
**Date:** December 2025  
**Status:** ✅ Deployed & Live  
**Author:** Muhammad Mashood  

---

## 📋 Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Product Overview](#2-product-overview)
3. [Core Features](#3-core-features)
4. [Technical Architecture](#4-technical-architecture)
5. [Data Models](#5-data-models)
6. [User Experience (UX) Flows](#6-user-experience-ux-flows)
7. [Security Implementation](#7-security-implementation)
8. [Firebase Setup Guide](#8-firebase-setup-guide)
9. [Edge Cases & Error Handling](#9-edge-cases--error-handling)
10. [Deployment](#10-deployment)
11. [Cost Analysis](#11-cost-analysis)
12. [Future Enhancements](#12-future-enhancements)

---

## 1. Executive Summary

### 1.1 What is DuoTasks?

DuoTasks is a lightweight, real-time project management tool designed specifically for **two people** to collaborate on tasks. It provides a Kanban-style board with task cards, relationship linking, threaded comments, and a visual tree view — all in a **single HTML file** deployed on GitHub Pages with Firebase as the backend.

### 1.2 Key Highlights

| Aspect | Description |
|--------|-------------|
| **Users** | 2 (you + friend) |
| **Cost** | $0 (Firebase free tier) |
| **Hosting** | GitHub Pages (free) |
| **Tech Stack** | Single HTML file + Firebase |
| **Real-time** | Yes, instant sync |
| **Authentication** | Google Sign-in with email whitelist |

### 1.3 Problem Solved

Most project management tools (Jira, Asana, Trello) are either:
- Overkill for 2 people
- Require paid plans for useful features
- Need server infrastructure

DuoTasks solves this by providing a **minimal, free, self-hosted** solution with professional features.

---

## 2. Product Overview

### 2.1 Target Users

- Small teams of 2 people
- Side project collaborators
- Friends working on shared projects
- Couples managing household tasks
- Study partners tracking assignments

### 2.2 Design Principles

| Principle | Implementation |
|-----------|----------------|
| **Simplicity** | Single HTML file, no build process |
| **Real-time** | Firebase Realtime Database |
| **Security** | Email-based whitelist |
| **Zero Cost** | Free tiers only |
| **Modern UX** | Dark theme, smooth animations |

### 2.3 User Interface Overview

```
┌─────────────────────────────────────────────────────────────────┐
│  🟦 DuoTasks    [🔍 Search]    [🟢 Synced]    [👤 User]  [+ New]│
├─────────────┬───────────────────────────────────────────────────┤
│             │                                                   │
│  FILTERS    │   [Board]  [Tree View]                           │
│  ○ All      │                                                   │
│  ○ My Tasks │   ┌─────────┐  ┌─────────┐  ┌─────────┐          │
│  ○ Due Soon │   │ 🔵 TODO │  │ 🟠 IN   │  │ 🟢 DONE │          │
│             │   │         │  │ PROGRESS│  │         │          │
│  TEAM       │   │ [Card]  │  │ [Card]  │  │ [Card]  │          │
│  🟢 You     │   │ [Card]  │  │         │  │         │          │
│  🟢 Friend  │   │         │  │         │  │         │          │
│             │   │ [+ Add] │  │ [+ Add] │  │         │          │
│  STATS      │   └─────────┘  └─────────┘  └─────────┘          │
│  Todo: 2    │                                                   │
│  Active: 1  │                                                   │
│  Done: 1    │                                                   │
│             │                                                   │
└─────────────┴───────────────────────────────────────────────────┘
```

---

## 3. Core Features

### 3.1 Task Cards

Each task is represented as a card with the following properties:

#### 3.1.1 Task Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | string | Auto | Unique identifier (Firebase push key) |
| `title` | string | Yes | Task name |
| `description` | string (HTML) | No | Rich-text description |
| `assignedTo` | string (UID) | Yes | User responsible for task |
| `status` | enum | Yes | `todo`, `in-progress`, `done` |
| `createdAt` | timestamp | Auto | Creation time |
| `createdBy` | string (UID) | Auto | Creator's user ID |
| `dueDate` | timestamp | No | Optional deadline |
| `links` | object | Auto | Relationships to other tasks |
| `comments` | object | Auto | Threaded comments |

#### 3.1.2 Task Card UI Elements

```
┌─────────────────────────────────┐
│ Task Title               #abc12 │  ← ID badge
├─────────────────────────────────┤
│ Description preview text that   │  ← 2-line clamp
│ gets truncated if too long...   │
├─────────────────────────────────┤
│ [🔗 2] [💬 3] [📅 2d]      👤   │  ← Badges + Avatar
└─────────────────────────────────┘
```

#### 3.1.3 Task Card Badges

| Badge | Meaning |
|-------|---------|
| 🔗 2 | Has 2 linked tasks |
| 💬 3 | Has 3 comments |
| 📅 2d | Due in 2 days |
| 📅 Overdue | Past due date (red) |

#### 3.1.4 Task Operations

| Operation | Trigger | Action |
|-----------|---------|--------|
| **Create** | "+ New Task" button or "+ Add" in column | Opens create modal |
| **View** | Click on card | Opens detail modal |
| **Edit** | "Edit" button in detail modal | Opens edit modal |
| **Delete** | "Delete" button in detail modal | Confirmation → removes task |
| **Move** | Drag & drop | Changes status |

---

### 3.2 Kanban Board

#### 3.2.1 Columns

| Column | Status Value | Color |
|--------|--------------|-------|
| To Do | `todo` | 🔵 Blue |
| In Progress | `in-progress` | 🟠 Orange |
| Done | `done` | 🟢 Green |

#### 3.2.2 Drag & Drop

- Tasks can be dragged between columns
- Status updates immediately in Firebase
- Visual feedback during drag (rotation, opacity)
- Real-time sync — other user sees change instantly

#### 3.2.3 Column Features

- Task count badge
- Scrollable if many tasks
- "+ Add Task" button (except Done column)
- Empty state message when no tasks

---

### 3.3 Card Linking (Relationships)

#### 3.3.1 Link Types

| Type | Meaning | Reverse Type |
|------|---------|--------------|
| `depends-on` | This task needs another to finish first | `blocks` |
| `blocks` | This task blocks another | `depends-on` |
| `parent` | This is a parent/epic task | `child` |
| `child` | This is a subtask | `parent` |
| `related-to` | General relationship | `related-to` |

#### 3.3.2 Link Properties

| Property | Description |
|----------|-------------|
| **Bidirectional** | Creating A→B automatically creates B→A |
| **Navigable** | Click linked task to open it |
| **Removable** | Remove from either side |
| **Visual** | Shown in Tree View with color coding |

#### 3.3.3 Link UI in Task Detail

```
┌─────────────────────────────┐
│ 🔗 Linked Tasks             │
├─────────────────────────────┤
│ [DEPENDS-ON] Setup Database │  ← Clickable
│ [RELATED-TO] Design Mockups │
├─────────────────────────────┤
│ Type: [Depends On    ▼]     │
│ Task: [Select task   ▼]     │
│ [+ Add Link]                │
└─────────────────────────────┘
```

---

### 3.4 Tree / Flow View

#### 3.4.1 Purpose

Visualize all tasks and their relationships in a graph layout.

#### 3.4.2 Visual Elements

| Element | Representation |
|---------|----------------|
| Tasks | Rectangular nodes |
| Parent/Child links | Green solid lines |
| Dependency links | Orange solid lines |
| Related links | Blue dashed lines |
| Current task | Purple border (when opened from detail) |

#### 3.4.3 Tree View Controls

| Control | Action |
|---------|--------|
| 🔍+ Zoom In | Increase zoom by 20% |
| 🔍- Zoom Out | Decrease zoom by 20% |
| 🔄 Reset | Reset to default zoom/position |
| Mouse drag | Pan the view |
| Click node | Open task detail |

#### 3.4.4 Tree View Layout

```
        ┌──────────┐
        │  Task A  │
        │  🔵 todo │
        └────┬─────┘
             │ (green = parent/child)
     ┌───────┴───────┐
     │               │
┌────▼────┐    ┌────▼────┐
│ Task B  │····│ Task C  │  (blue dashed = related)
│ 🟠 prog │    │ 🟢 done │
└─────────┘    └─────────┘
```

---

### 3.5 Comments System

#### 3.5.1 Comment Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | string | Unique identifier |
| `authorId` | string | Commenter's UID |
| `text` | string | Comment content |
| `timestamp` | number | When posted |
| `parentId` | string/null | For threaded replies |

#### 3.5.2 Comment Features

| Feature | Description |
|---------|-------------|
| **Threaded replies** | Reply to specific comments |
| **Author info** | Shows name + avatar |
| **Timestamps** | Relative time (e.g., "5m ago") |
| **Delete** | Only author can delete their comment |
| **Persistent** | Comments remain even if task status changes |

#### 3.5.3 Comment UI

```
┌─────────────────────────────────────────┐
│ 💬 Comments                   3 comments│
├─────────────────────────────────────────┤
│ [Your Avatar] [Write a comment...]      │
│                              [Post]     │
├─────────────────────────────────────────┤
│ 👤 Alice                        5m ago  │
│ ┌─────────────────────────────────────┐ │
│ │ Great progress on this task!        │ │
│ └─────────────────────────────────────┘ │
│ [Reply] [Delete]                        │
│                                         │
│    ↳ 👤 Bob                     2m ago  │
│      ┌───────────────────────────────┐  │
│      │ Thanks! Almost done.          │  │
│      └───────────────────────────────┘  │
└─────────────────────────────────────────┘
```

---

### 3.6 User Management

#### 3.6.1 Authentication

| Method | Provider |
|--------|----------|
| Google Sign-in | Firebase Authentication |

#### 3.6.2 User Properties (stored in Firebase)

| Property | Description |
|----------|-------------|
| `uid` | Firebase Auth UID |
| `email` | Google email |
| `name` | Display name |
| `photoURL` | Profile picture |
| `online` | Boolean status |
| `lastSeen` | Timestamp |

#### 3.6.3 Online/Offline Status

- **Online**: User has the app open
- **Offline**: User closed tab or lost connection
- **Real-time**: Status updates instantly for other user
- **Automatic**: Uses Firebase presence system

---

### 3.7 Filters & Search

#### 3.7.1 Quick Filters

| Filter | Shows |
|--------|-------|
| **All Tasks** | Every task |
| **Assigned to Me** | Tasks where `assignedTo === currentUser.uid` |
| **Due Soon** | Tasks due within 7 days |

#### 3.7.2 Search

- Searches task titles and descriptions
- Real-time filtering as you type
- Case-insensitive
- Keyboard shortcut: `Ctrl+K`

---

### 3.8 Statistics

Displayed in sidebar:

| Stat | Calculation |
|------|-------------|
| To Do | Count of `status === 'todo'` |
| In Progress | Count of `status === 'in-progress'` |
| Done | Count of `status === 'done'` |
| Total | All tasks count |

---

### 3.9 Real-time Sync

#### 3.9.1 Sync Indicator

| Status | Meaning |
|--------|---------|
| 🟢 Synced | Connected to Firebase |
| 🔄 Connecting... | Establishing connection |

#### 3.9.2 What Syncs in Real-time

- Task creation/updates/deletion
- Comments
- Status changes (drag & drop)
- User online/offline status
- Link additions/removals

---

## 4. Technical Architecture

### 4.1 High-Level Architecture

```
┌─────────────────┐     ┌─────────────────┐
│   User 1        │     │   User 2        │
│   (Browser)     │     │   (Browser)     │
└────────┬────────┘     └────────┬────────┘
         │                       │
         │   WebSocket           │   WebSocket
         │                       │
         └───────────┬───────────┘
                     │
         ┌───────────▼───────────┐
         │   GitHub Pages        │
         │   (Static HTML)       │
         └───────────┬───────────┘
                     │
         ┌───────────▼───────────┐
         │   Firebase            │
         │  ┌─────────────────┐  │
         │  │ Authentication  │  │
         │  │ (Google OAuth)  │  │
         │  └─────────────────┘  │
         │  ┌─────────────────┐  │
         │  │ Realtime DB     │  │
         │  │ (Tasks, Users)  │  │
         │  └─────────────────┘  │
         └───────────────────────┘
```

### 4.2 Tech Stack

| Layer | Technology |
|-------|------------|
| **Frontend** | Vanilla HTML/CSS/JavaScript |
| **Styling** | CSS Custom Properties (variables) |
| **Backend** | Firebase Realtime Database |
| **Authentication** | Firebase Auth (Google) |
| **Hosting** | GitHub Pages |
| **Real-time** | Firebase WebSocket |

### 4.3 File Structure

```
Single File: index.html
│
├── <style>
│   ├── CSS Variables (theming)
│   ├── Login Screen styles
│   ├── Header styles
│   ├── Sidebar styles
│   ├── Board/Column styles
│   ├── Task Card styles
│   ├── Modal styles
│   ├── Tree View styles
│   ├── Comments styles
│   └── Utility classes
│
├── <body>
│   ├── Login Screen
│   ├── Main App Container
│   │   ├── Header
│   │   ├── Sidebar
│   │   └── Board Container
│   ├── Create/Edit Modal
│   ├── Task Detail Modal
│   ├── Delete Confirmation Modal
│   └── Toast Container
│
└── <script>
    ├── Firebase Config
    ├── Allowed Users Array
    ├── App State
    ├── Firebase Initialization
    ├── Authentication Functions
    ├── Realtime Listeners
    ├── CRUD Operations
    ├── Linking Functions
    ├── Comments Functions
    ├── Rendering Functions
    ├── Drag & Drop
    ├── Modal Handlers
    ├── Tree View Logic
    ├── Filters & Search
    └── Keyboard Shortcuts
```

### 4.4 State Management

```javascript
// Client-side state (synced with Firebase)
let currentUser = null;      // Logged-in user object
let tasks = {};              // All tasks (keyed by ID)
let teamMembers = {};        // All users (keyed by UID)
let currentFilter = 'all';   // Active filter
let currentView = 'board';   // 'board' or 'tree'
let currentTaskId = null;    // Task open in detail modal

// Tree view state
let treeZoom = 1;
let treeOffset = { x: 0, y: 0 };
```

---

## 5. Data Models

### 5.1 Firebase Database Structure

```
root/
├── tasks/
│   └── {taskId}/
│       ├── id: string
│       ├── title: string
│       ├── description: string (HTML)
│       ├── assignedTo: string (UID)
│       ├── status: "todo" | "in-progress" | "done"
│       ├── createdAt: timestamp
│       ├── createdBy: string (UID)
│       ├── dueDate: timestamp | null
│       ├── links/
│       │   └── {linkedTaskId}/
│       │       └── type: string
│       └── comments/
│           └── {commentId}/
│               ├── id: string
│               ├── authorId: string (UID)
│               ├── text: string
│               ├── timestamp: number
│               └── parentId: string | null
│
└── users/
    └── {uid}/
        ├── uid: string
        ├── email: string
        ├── name: string
        ├── photoURL: string
        ├── online: boolean
        └── lastSeen: timestamp
```

### 5.2 Task Schema

```typescript
interface Task {
  id: string;                    // Firebase push key
  title: string;                 // Required, user input
  description: string;           // HTML from rich editor
  assignedTo: string;            // User UID
  status: 'todo' | 'in-progress' | 'done';
  createdAt: number;             // Server timestamp
  createdBy: string;             // Creator's UID
  dueDate: number | null;        // Optional timestamp
  links: {
    [taskId: string]: {
      type: 'depends-on' | 'blocks' | 'parent' | 'child' | 'related-to';
    }
  };
  comments: {
    [commentId: string]: Comment;
  };
}
```

### 5.3 User Schema

```typescript
interface User {
  uid: string;           // Firebase Auth UID
  email: string;         // Google email
  name: string;          // Display name
  photoURL: string;      // Profile picture URL
  online: boolean;       // Presence status
  lastSeen: number;      // Last activity timestamp
}
```

### 5.4 Comment Schema

```typescript
interface Comment {
  id: string;                  // Unique ID
  authorId: string;            // Commenter's UID
  text: string;                // Comment content
  timestamp: number;           // When posted
  parentId: string | null;     // Parent comment (for threads)
}
```

### 5.5 Link Schema

```typescript
interface Link {
  type: 'depends-on' | 'blocks' | 'parent' | 'child' | 'related-to';
}

// Stored as: tasks/{taskId}/links/{linkedTaskId}/type
```

---

## 6. User Experience (UX) Flows

### 6.1 First-Time User Flow

```
1. User visits GitHub Pages URL
           ↓
2. Sees login screen with "Sign in with Google"
           ↓
3. Clicks button → Google OAuth popup
           ↓
4. Selects their Google account
           ↓
5. Firebase checks if email is whitelisted
           ↓
   ├── ✅ Yes → Show main app (empty board)
   └── ❌ No  → Show "Access denied" error
```

### 6.2 Creating a Task

```
1. Click "+ New Task" or "+ Add" in column
           ↓
2. Modal opens with form:
   - Title (required)
   - Description (rich text)
   - Assignee (dropdown)
   - Status (dropdown)
   - Due Date (date picker)
           ↓
3. Fill in details → Click "Save Task"
           ↓
4. Task saved to Firebase
           ↓
5. Board updates instantly (both users)
           ↓
6. Toast: "Task created successfully"
```

### 6.3 Moving a Task (Drag & Drop)

```
1. Click and hold a task card
           ↓
2. Card shows drag state (rotated, semi-transparent)
           ↓
3. Drag to another column
           ↓
4. Release → Card drops
           ↓
5. Status updates in Firebase
           ↓
6. Other user sees change instantly
           ↓
7. Toast: "Task moved"
```

### 6.4 Linking Tasks

```
1. Open task detail modal
           ↓
2. Scroll to "🔗 Linked Tasks" section
           ↓
3. Select link type from dropdown:
   - Depends On
   - Related To
   - Parent Task
   - Child Task
           ↓
4. Select target task from dropdown
           ↓
5. Click "+ Add Link"
           ↓
6. Bidirectional link created:
   - Task A → "depends-on" → Task B
   - Task B → "blocks" → Task A
           ↓
7. Both tasks show the link
```

### 6.5 Viewing Tree View

```
1. Click "Tree View" tab
           ↓
2. All tasks render as nodes
           ↓
3. Links shown as colored lines:
   - Green = parent/child
   - Orange = dependencies
   - Blue dashed = related
           ↓
4. Controls available:
   - Zoom in/out
   - Pan (drag background)
   - Click node → opens task detail
           ↓
5. Reset button → default view
```

### 6.6 Adding a Comment

```
1. Open task detail modal
           ↓
2. Scroll to "💬 Comments" section
           ↓
3. Type in comment box
           ↓
4. Click "Post"
           ↓
5. Comment saved with:
   - Your name/avatar
   - Timestamp
           ↓
6. Other user sees it instantly
```

### 6.7 Replying to a Comment

```
1. Find a comment
           ↓
2. Click "Reply"
           ↓
3. Reply box expands below comment
           ↓
4. Type reply → Click "Reply" button
           ↓
5. Reply appears indented under original
```

---

## 7. Security Implementation

### 7.1 Security Layers

```
┌─────────────────────────────────────────────┐
│  Layer 1: Firebase Auth                     │
│  - Must sign in with Google                 │
│  - Gets verified identity                   │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│  Layer 2: Client-side Check                 │
│  - ALLOWED_USERS array in HTML              │
│  - Checks email before showing app          │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│  Layer 3: Firebase Security Rules           │
│  - Server-side enforcement                  │
│  - Blocks all unauthorized requests         │
│  - Cannot be bypassed                       │
└─────────────────────────────────────────────┘
```

### 7.2 Client-Side Protection

```javascript
// In HTML file
const ALLOWED_USERS = [
    "user1@gmail.com",
    "user2@gmail.com"
];

// Auth state change handler
auth.onAuthStateChanged(user => {
    if (user) {
        if (!ALLOWED_USERS.includes(user.email)) {
            showError("Access denied");
            auth.signOut();
            return;
        }
        // Proceed to app...
    }
});
```

### 7.3 Server-Side Protection (Firebase Rules)

```json
{
  "rules": {
    ".read": "auth != null && (auth.token.email === 'user1@gmail.com' || auth.token.email === 'user2@gmail.com')",
    ".write": "auth != null && (auth.token.email === 'user1@gmail.com' || auth.token.email === 'user2@gmail.com')"
  }
}
```

### 7.4 What Each Layer Stops

| Attack | Layer 1 | Layer 2 | Layer 3 |
|--------|---------|---------|---------|
| Unauthenticated access | ✅ Blocked | — | ✅ Blocked |
| Wrong Google account | — | ✅ Blocked | ✅ Blocked |
| Direct API call with stolen keys | — | — | ✅ Blocked |
| Modified HTML | — | ❌ Bypassed | ✅ Blocked |

**Layer 3 (Firebase Rules) is the ultimate protection** — even if someone bypasses the client-side checks, the server blocks them.

### 7.5 Why Firebase Keys Are Safe to Expose

| Concern | Reality |
|---------|---------|
| "Someone sees my API key" | Key only identifies the project, doesn't grant access |
| "Someone makes API calls" | Firebase Rules block unauthorized requests |
| "I'll get billed" | Blocked requests don't count toward quota |

---

## 8. Firebase Setup Guide

### 8.1 Prerequisites

- Google account
- GitHub account

### 8.2 Step-by-Step Setup

#### Step 1: Create Firebase Project (2 min)

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click "Create a project"
3. Enter project name (e.g., `duotasks`)
4. Disable Google Analytics (not needed)
5. Click "Create project"
6. Wait for creation → Click "Continue"

#### Step 2: Add Web App (1 min)

1. On project overview, click Web icon (`</>`)
2. Enter app nickname: `DuoTasks Web`
3. Don't check "Firebase Hosting"
4. Click "Register app"
5. **Copy the config object** — you'll need it later:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "project.firebaseapp.com",
  databaseURL: "https://project-default-rtdb.firebaseio.com",
  projectId: "project",
  storageBucket: "project.appspot.com",
  messagingSenderId: "123456",
  appId: "1:123456:web:abcdef"
};
```

6. Click "Continue to console"

#### Step 3: Enable Google Authentication (1 min)

1. Left sidebar → "Build" → "Authentication"
2. Click "Get started"
3. "Sign-in method" tab → Click "Google"
4. Toggle "Enable" to ON
5. Select your email as support email
6. Click "Save"

#### Step 4: Create Realtime Database (2 min)

1. Left sidebar → "Build" → "Realtime Database"
2. Click "Create Database"
3. Choose location (closest to you)
4. Select "Start in test mode"
5. Click "Enable"

#### Step 5: Set Security Rules (2 min)

1. In Realtime Database → "Rules" tab
2. Delete everything, paste:

```json
{
  "rules": {
    ".read": "auth != null && (auth.token.email === 'YOUR_EMAIL@gmail.com' || auth.token.email === 'FRIEND_EMAIL@gmail.com')",
    ".write": "auth != null && (auth.token.email === 'YOUR_EMAIL@gmail.com' || auth.token.email === 'FRIEND_EMAIL@gmail.com')"
  }
}
```

3. Replace emails with actual addresses
4. Click "Publish"

#### Step 6: Configure HTML File (1 min)

1. Open `duotasks-firebase.html`
2. Find `firebaseConfig` object, replace with your config
3. Find `ALLOWED_USERS` array, add your emails:

```javascript
const ALLOWED_USERS = [
    "your-email@gmail.com",
    "friend-email@gmail.com"
];
```

4. Save file

#### Step 7: Add GitHub Pages Domain (1 min)

1. Firebase Console → "Build" → "Authentication"
2. "Settings" tab
3. Scroll to "Authorized domains"
4. Click "Add domain"
5. Enter: `yourusername.github.io`
6. Click "Add"

#### Step 8: Deploy to GitHub Pages (2 min)

1. Create new GitHub repository
2. Upload HTML file as `index.html`
3. Go to Settings → Pages
4. Source: `main` branch, `/ (root)`
5. Save → Wait 1 minute
6. Visit: `https://yourusername.github.io/reponame/`

### 8.3 Configuration Checklist

| Item | Location | Status |
|------|----------|--------|
| Firebase project | Firebase Console | ☐ |
| Google Auth enabled | Authentication | ☐ |
| Database created | Realtime Database | ☐ |
| Security rules set | Database → Rules | ☐ |
| Config in HTML | `firebaseConfig` | ☐ |
| Emails in HTML | `ALLOWED_USERS` | ☐ |
| Domain authorized | Auth → Settings | ☐ |
| Deployed to GitHub | GitHub Pages | ☐ |

---

## 9. Edge Cases & Error Handling

### 9.1 Task Deletion with Links

**Scenario:** User deletes a task that has links to other tasks.

**Handling:**
1. Show warning modal listing linked tasks
2. On confirm, remove links from all connected tasks
3. Delete the task
4. Toast: "Task deleted"

```javascript
async function deleteTask(taskId) {
    const task = tasks[taskId];
    
    // Remove links from other tasks
    if (task.links) {
        for (const linkedId of Object.keys(task.links)) {
            await database.ref(`tasks/${linkedId}/links/${taskId}`).remove();
        }
    }
    
    // Delete task
    await database.ref(`tasks/${taskId}`).remove();
}
```

### 9.2 Circular Dependencies

**Scenario:** User tries to create A → depends-on → B when B already depends on A.

**Handling:**
1. Before adding link, traverse dependency chain
2. If target task eventually depends on source → block
3. Show toast: "Cannot create circular dependency!"

```javascript
function wouldCreateCircular(sourceId, targetId, linkType) {
    if (linkType !== 'depends-on') return false;
    
    const visited = new Set();
    const queue = [targetId];
    
    while (queue.length > 0) {
        const current = queue.shift();
        if (current === sourceId) return true;  // Circular!
        if (visited.has(current)) continue;
        visited.add(current);
        
        const task = tasks[current];
        if (task?.links) {
            Object.entries(task.links)
                .filter(([, link]) => link.type === 'depends-on')
                .forEach(([id]) => queue.push(id));
        }
    }
    return false;
}
```

### 9.3 Orphan Tasks

**Scenario:** Task has no links (standalone).

**Handling:** Completely normal — tasks don't require links. They appear in board and tree view independently.

### 9.4 Network Disconnection

**Scenario:** User loses internet connection.

**Handling:**
1. Sync indicator changes to "🔄 Connecting..."
2. Firebase SDK queues local changes
3. When connection restored → auto-sync
4. Indicator returns to "🟢 Synced"

### 9.5 Unauthorized Access Attempt

**Scenario:** User not in whitelist tries to sign in.

**Handling:**
1. Google OAuth succeeds (they have valid Google account)
2. App checks `ALLOWED_USERS` array
3. Email not found → show error: "Access denied. [email] is not authorized."
4. Sign out automatically
5. Even if they bypass client check, Firebase Rules block all database access

### 9.6 Duplicate Links

**Scenario:** User tries to link same two tasks twice.

**Handling:**
1. Check if link already exists
2. If yes → show toast: "Link already exists!"
3. Don't create duplicate

### 9.7 Missing Database URL

**Scenario:** User forgets to add `databaseURL` in config.

**Handling:**
1. App shows "🔄 Connecting..." indefinitely
2. Console shows Firebase error
3. Solution: Copy correct URL from Firebase Console

---

## 10. Deployment

### 10.1 GitHub Pages Deployment

#### Option A: Web Interface

1. Go to github.com → New repository
2. Name it (e.g., `duotasks`)
3. Make it **Public**
4. Create repository
5. Click "uploading an existing file"
6. Rename `duotasks-firebase.html` to `index.html`
7. Drag file → Commit changes
8. Settings → Pages → Source: `main`, `/ (root)`
9. Save → Wait 1 minute
10. Access at: `https://username.github.io/duotasks/`

#### Option B: Git CLI

```bash
git init
mv duotasks-firebase.html index.html
git add index.html
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/USERNAME/duotasks.git
git push -u origin main
# Then enable Pages in Settings
```

### 10.2 Updating the App

1. Edit `index.html` locally
2. Push to GitHub:
   ```bash
   git add index.html
   git commit -m "Update"
   git push
   ```
3. Wait ~1 minute for GitHub Pages to refresh

### 10.3 Custom Domain (Optional)

1. Buy domain (e.g., `duotasks.com`)
2. In repo Settings → Pages → Custom domain
3. Add DNS records as instructed
4. Add domain to Firebase authorized domains

---

## 11. Cost Analysis

### 11.1 Firebase Free Tier (Spark Plan)

| Resource | Free Limit | DuoTasks Usage |
|----------|------------|----------------|
| Realtime DB Storage | 1 GB | ~50 KB |
| Realtime DB Downloads | 10 GB/month | ~10 MB |
| Simultaneous Connections | 100 | 2 |
| Auth Users | Unlimited | 2 |

### 11.2 Usage Estimates

| Action | Data Size | Daily Est. | Monthly Est. |
|--------|-----------|------------|--------------|
| Page load | ~10 KB | 20 loads = 200 KB | 6 MB |
| Create task | ~1 KB | 10 tasks = 10 KB | 300 KB |
| Update task | ~0.5 KB | 50 updates = 25 KB | 750 KB |
| Add comment | ~0.2 KB | 20 comments = 4 KB | 120 KB |
| **Total** | — | ~240 KB/day | ~7.2 MB/month |

### 11.3 Cost Breakdown

| Service | Plan | Cost |
|---------|------|------|
| Firebase | Spark (free) | $0 |
| GitHub Pages | Free | $0 |
| Custom Domain | Optional | $10-15/year |
| **Total** | — | **$0/month** |

### 11.4 When Would You Need to Pay?

You'd need to **manually upgrade** to Blaze (pay-as-you-go) if you:
- Have 100+ simultaneous users
- Store more than 1 GB
- Transfer more than 10 GB/month

**For 2 users:** Impossible to hit these limits.

---

## 12. Future Enhancements

### 12.1 Near-Term (Low Effort)

| Feature | Description | Effort |
|---------|-------------|--------|
| **Tags/Labels** | Add colored tags to tasks | 2 hours |
| **Sort Options** | Sort by date, priority, assignee | 1 hour |
| **Task Priority** | High/Medium/Low priority field | 1 hour |
| **Keyboard Shortcuts** | More shortcuts (e.g., `E` to edit) | 1 hour |
| **Dark/Light Toggle** | Theme switcher | 2 hours |

### 12.2 Medium-Term (Moderate Effort)

| Feature | Description | Effort |
|---------|-------------|--------|
| **Activity Log** | Track all changes with timestamps | 4 hours |
| **Due Date Reminders** | Browser notifications | 3 hours |
| **File Attachments** | Firebase Storage integration | 4 hours |
| **Markdown Support** | Full markdown in descriptions | 3 hours |
| **Export to JSON/CSV** | Download task data | 2 hours |

### 12.3 Long-Term (Higher Effort)

| Feature | Description | Effort |
|---------|-------------|--------|
| **Mobile App** | React Native or PWA | 20+ hours |
| **Calendar View** | Visual calendar with due dates | 8 hours |
| **Time Tracking** | Log time spent on tasks | 6 hours |
| **Email Notifications** | Firebase Functions for alerts | 6 hours |
| **Multiple Projects** | Separate boards/workspaces | 10 hours |

### 12.4 Not Planned (Out of Scope)

- Enterprise features (SSO, audit logs)
- Team size > 5 people
- Paid features
- Complex permissions

---

## 📄 Document History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Dec 2025 | Initial release |

---

## 🙏 Acknowledgments

- **Firebase** — Free backend infrastructure
- **GitHub Pages** — Free hosting
- **Google** — OAuth authentication

---

*This PRD documents DuoTasks as of December 2025. The product is fully functional and deployed.
https://mashoodawan.github.io/TaskManagementTool/ *
