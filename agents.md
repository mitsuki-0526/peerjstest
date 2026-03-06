# Baboot! AI Agent Specification

This document provides technical guidelines and specifications for AI agents collaborating on the "Baboot!" project.

## Project Overview
"Baboot!" is a real-time multiplayer quiz game designed for school environments. It uses Firebase Realtime Database for communication between a host (Teacher) and participants (Students).

## Key Files
- `teacher.html`: Hosting dashboard, quiz editor, and projector view.
- `index.html`: Student participant client.
- `Code.gs`: (Legacy/Optional) Google Apps Script for online quiz data management.

## Technical Stack
- **Languages**: HTML5, Vanilla CSS, Vanilla JavaScript.
- **Backend**: Firebase Realtime Database (RTDB).
- **Communication Protocol**: Data-driven state synchronization using RTDB.

## Firebase Realtime Database Schema

### Root: `/rooms/{roomId}/`
`roomId` is prefixed with `baboot-` (e.g., `baboot-123456`).

| Path | Description | Data Structure |
| :--- | :--- | :--- |
| `state` | Current room status | `LOBBY`, `QUESTION`, `RESULT`, etc. |
| `broadcast` | Master command node | `{type, ...data, _timestamp}` |
| `students/{studentId}` | Registered participants | `{name, timestamp}` |
| `answers/{studentId}` | Submitted answers | `{choice, timeTaken}` |
| `results/{studentId}` | Feedback for students | `{type: "RESULT", isCorrect, points, bonus}` |
| `kicks/{studentId}` | Forceful kick signal | `{timestamp}` |

### Global Settings
- `/system/force_reset`: A timestamp. If `resetTime > pageLoadTime`, clients must disconnect and reload.

## Communication Protocols

### Broadcast types
- `PRE_QUESTION`: Syncs question text and options to students.
- `QUESTION_START`: Triggers the timer on student devices.
- `RANKING`: Sends top 5 leaderboard data.
- `FINAL_RANKING`: Sends final podium results.
- `END`: Signals game termination.
- `NEXT`: Resets student state for the next question.

## Implementation Rules & Security

### 1. Cross-Site Scripting (XSS) Prevention
- **Rule**: Never use `innerHTML` or `textContent` directly with user-provided strings (Student names, Quiz content).
- **Tool**: Always use the `escapeHtml(str)` helper function or `document.createTextNode()`.

### 2. Student Identification
- **Rule**: Identification must rely on `studentId` (a unique random string), **not** the `name` property, as names can be duplicated.
- **Check**: When rendering ranking lists, check `p.studentId === studentId` to highlight "Me".

### 3. Connection Slot Management
- **Rule**: Minimize concurrent Firebase connections to stay within free tier limits (100 concurrent).
- **Methods**: Use `db.goOffline()` when on the dashboard or after the game ends. Use `db.goOnline()` only when active communication is required.

### 4. Cleanup Procedures
- **Teacher**: When closing a room, wait for a short delay (e.g., 1500ms) after broadcasting `END` before calling `roomRef.remove()` to ensure delivery.
- **Student**: Use `onDisconnect().remove()` for the student node in RTDB.

---
*Note: This specification is maintained by the AI agents. Please update as the schema evolves.*
