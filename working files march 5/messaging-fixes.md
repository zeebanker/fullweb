# Messaging System — Applied Fixes
**File:** `loonie-bird.html`
**Date:** 2026-03-05

---

## Bug 2 — Crash on empty conversation · `getConversations()`
Added a guard after `lastMsg` is assigned. If `convMessages` is empty, `lastMsg` is `undefined` and the push is skipped.
```js
const lastMsg = convMessages[convMessages.length - 1];
if (!lastMsg) return; // ← added
```

---

## Bug 4 — Inverted read logic for students · `showStudentMessagesView()`
The second argument to `markConversationRead` was `studentName`, which caused the student's own outgoing messages to be marked read instead of the teacher's incoming ones.
```js
// Before
markConversationRead(studentName, studentName);

// After
markConversationRead(studentName, 'teacher');
```

---

## Bug 5 — Apostrophe in student name breaks onclick · `showMessagesView()`
Removed inline `onclick` string interpolation from conversation sidebar items. Names are now stored in a `data-conv-index` attribute and a single delegated listener on `#conversationList` handles clicks.
```js
// Before — breaks on O'Brien, etc.
html += '<div ... onclick="selectConversation(\'' + conv.id + '\')">';

// After
html += '<div ... data-conv-index="' + i + '">';

// Delegated listener attached after render
convListEl.addEventListener('click', function(e) {
    const item = e.target.closest('.conversation-item');
    if (!item) return;
    const idx = parseInt(item.getAttribute('data-conv-index'));
    const convs = getConversations();
    if (convs[idx]) selectConversation(convs[idx].id);
});
```

---

## Bug 7 — Teacher can't initiate a conversation · `showMessagesView()`
Added a **+ New Message** button to the sidebar header. It calls `showNewMessagePicker()`, which lists all non-retired students and lets the teacher open a thread with any of them (showing "Existing thread" if one already exists).

**New functions added:**
- `showNewMessagePicker()` — renders student picker in modal
- `openNewConversation(firstName, lastName)` — sets `currentConversationId` and opens the messages view

---

## Bug 8 — Fragile active conversation highlight · `selectConversation()`
Replaced the `getAttribute('onclick').includes(...)` check with a `data-conv-index` lookup, consistent with the fix for Bug 5.
```js
// Before — fragile string match
el.classList.toggle('active', el.getAttribute('onclick').includes("'" + conversationId + "'"));

// After
const activeIdx = allConvs.findIndex(c => c.id === conversationId);
el.classList.toggle('active', el.getAttribute('data-conv-index') === String(activeIdx));
```

---

## Bug 9 — `onkeypress` deprecated · both compose textareas
Replaced `onkeypress` with `onkeydown` on both the teacher (`#messageInput`) and student (`#studentMessageInput`) compose textareas.

---

## Bug 10 — Focus not restored after sending · `sendTeacherMessage()`, `sendStudentMessage()`
After the thread refreshes, focus is returned to the compose textarea via a 50ms `setTimeout` (the delay allows the DOM to re-render first).
```js
setTimeout(() => {
    const inp = document.getElementById('messageInput');
    if (inp) inp.focus();
}, 50);
```

---

## Bug 11 — No empty state for students · `showStudentMessagesView()`
The empty state subtext was generic. Updated to prompt the student to send a message:
```
"Send your teacher a message below to get started."
```

---

## Bug 12 — No character limit on message inputs
Added `maxlength="2000"` to both the teacher (`#messageInput`) and student (`#studentMessageInput`) compose textareas.

---

## Bug 13 — No way to delete conversations · `selectConversation()`
Added a 🗑 Delete button to the teacher thread header. It calls `deleteConversation(conversationId)`, which shows a `confirm()` dialog, filters the message out of the `messages` array, clears `currentConversationId` if it matches, saves, toasts, and re-renders.

**New function added:**
```js
function deleteConversation(conversationId) {
    if (!confirm('Delete all messages with ' + conversationId + '? This cannot be undone.')) return;
    messages = messages.filter(m => m.conversationId !== conversationId);
    if (currentConversationId === conversationId) currentConversationId = null;
    saveData();
    toast('Conversation deleted');
    showMessagesView();
}
```

---

## Bug 14 — No read receipts · `selectConversation()`, `showStudentMessagesView()`
Added a read status indicator below each message timestamp. Uses the existing `msg.read` boolean.

- Teacher sent → **✓✓ Read** (teal) or **✓ Sent** (grey)
- Student sent → same logic, shown to student in their thread

```js
if (msg.from === 'teacher') {
    html += msg.read
        ? ' <span style="color:var(--teal-500);">✓✓ Read</span>'
        : ' <span style="color:var(--stone-400);">✓ Sent</span>';
}
```

---

## Bug 15 — Apostrophe in seed data
Escaped the apostrophe in the demo message `"Don't forget your recital..."` to prevent issues if the seed data is ever serialised or re-parsed.

---

## Confirmed Already Fixed / Not Regressed

| # | Bug | Status |
|---|-----|--------|
| 1 | Messages not saved to localStorage | ✅ Already present in merged file |
| 3 | Null crash in `sendTeacherMessage()` | ✅ Guard already present |
| 6 | Unread badge missing on login | ✅ `updateMessageBadges()` called in `loginAsTeacher()` and `showStudentDashboard()` |
