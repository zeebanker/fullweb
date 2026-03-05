# Messaging System — Bug Report
**File:** `loonie-bird.html`
**Scope:** In-app messaging system only

---

## Origin Key
- 🔴 **Regression** — introduced during our file merge; was working in `index.html`
- 🟠 **Pre-existing** — present in `index.html`, carried through unchanged

---

## Critical

### 🔴 Messages not saved to localStorage
`saveData()` and `loadData()` in the merged file do not include `messages` or `nextMessageId`. All messages and read state are lost on page refresh. The original `index.html` had both fields correctly included in the data object.

**Fix:** Add `messages` and `nextMessageId` to the save/load data object alongside `lessons`, `swapRequests`, etc.

---

### 🟠 Crash if a conversation has zero messages
In `getConversations()`, `lastMsg` is assigned `convMessages[convMessages.length - 1]`. If `convMessages` is empty, `lastMsg` is `undefined` and `lastMsg.text` throws, taking down the entire messages view.

**Fix:** Skip or exclude any conversation ID that has no messages in the array.

---

### 🔴 Null crash on send with no conversation selected
In the original `index.html`, `sendTeacherMessage()` guards against a null `currentConversationId` before proceeding. That guard was dropped in the merge. Currently, if the teacher clicks Send with no conversation open, it calls `selectConversation(null)`, which filters messages against `null` and throws.

**Fix:** Add `if (!currentConversationId) return;` at the top of `sendTeacherMessage()`.

---

### 🟠 Read logic inverted for students
`showStudentMessagesView()` calls `markConversationRead(studentName, studentName)`. Inside `markConversationRead`, messages are marked read where `msg.from !== reader`. Because student messages have `from: studentName`, this marks the student's own *outgoing* messages as read instead of the teacher's *incoming* messages.

**Fix:** The second argument should be `'teacher'` so the function marks messages from the teacher as read — not the student's own.

---

## High

### 🟠 Apostrophe in student name breaks onclick
`showMessagesView()` builds sidebar items using string interpolation:
```js
onclick="selectConversation('" + conv.id + "')"
```
A student name containing `'` (e.g. `O'Brien`) produces malformed HTML and a JS syntax error that silently breaks the sidebar.

**Fix:** Use a `data-conv-id` attribute on each `.conversation-item` and a delegated event listener instead of an inline onclick.

---

### 🔴 Unread badge not shown on login
The original `index.html` calls `updateMessageBadges()` inside both `loginAsTeacher()` and `showStudentDashboard()`, so unread counts appear immediately on login. The merged file appears to have lost at least one of these call sites.

**Fix:** Verify that `updateMessageBadges()` is called at the end of both `loginAsTeacher()` and `showStudentDashboard()`.

---

### 🟠 No way for teacher to initiate a conversation
The teacher can only reply to students who have messaged first. There is no "New Message" button or student picker to open a blank thread.

**Fix:** Add a button that lets the teacher select a student from `studentProfiles` and creates a new thread.

---

## Medium

### 🟠 Active conversation highlight is fragile
After the infinite-loop fix, the sidebar active state is toggled by checking `el.getAttribute('onclick').includes("'" + conversationId + "'")`. This silently breaks for names with special characters and will stop working if the onclick format ever changes.

**Fix:** Set `data-conv-id` on each `.conversation-item` and match against that attribute directly.

---

### 🟠 `onkeypress` is deprecated
Both the teacher and student compose textareas use `onkeypress` to detect Enter. This event is deprecated, does not fire consistently on mobile, and does not work with IME input methods.

**Fix:** Replace with `onkeydown`.

---

### 🟠 Focus not restored after sending
After `sendTeacherMessage()` or `sendStudentMessage()` refreshes the thread, focus is not returned to the textarea. The user has to click back in to type a follow-up message.

**Fix:** Call `document.getElementById('messageInput')?.focus()` at the end of each send function, after the thread refreshes.

---

### 🟠 No empty state for students with no messages
If a student has never exchanged messages with the teacher, `showStudentMessagesView()` renders a blank thread area with no explanation or prompt.

**Fix:** Add an empty state block — e.g. "No messages yet. Send your teacher a message below."

---

## Low

### 🟠 No character limit on message input
Students and teachers can paste arbitrarily large text with no validation. No `maxlength` or character counter.

### 🟠 No way to delete messages or conversations
The teacher has no UI to remove a conversation or individual message.

### 🟠 No read receipts for students
Students cannot see whether the teacher has read their message.

### 🟠 Demo seed data references students not in `studentProfiles`
`"Sophia Garcia"` and `"Noah Wilson"` appear in the seed `messages` array but may not have matching entries in `studentProfiles` or `lessons`. Any code that tries to look up a profile from a conversation ID will fail silently for these names.

---

## Summary Table

| # | Bug | Origin | Severity |
|---|-----|--------|----------|
| 1 | Messages not saved to localStorage | 🔴 Regression | Critical |
| 2 | Crash on empty conversation in `getConversations()` | 🟠 Pre-existing | Critical |
| 3 | Null crash in `sendTeacherMessage()` | 🔴 Regression | Critical |
| 4 | Read logic inverted for students | 🟠 Pre-existing | Critical |
| 5 | Apostrophe in name breaks onclick | 🟠 Pre-existing | High |
| 6 | Unread badge missing on login | 🔴 Regression | High |
| 7 | No way to initiate a conversation | 🟠 Pre-existing | High |
| 8 | Fragile active conversation highlight | 🟠 Pre-existing | Medium |
| 9 | `onkeypress` deprecated | 🟠 Pre-existing | Medium |
| 10 | Focus not restored after sending | 🟠 Pre-existing | Medium |
| 11 | No empty state for students | 🟠 Pre-existing | Medium |
| 12 | No character limit on input | 🟠 Pre-existing | Low |
| 13 | No message/conversation deletion | 🟠 Pre-existing | Low |
| 14 | No read receipts | 🟠 Pre-existing | Low |
| 15 | Seed data references unknown students | 🟠 Pre-existing | Low |
