# Loonie Bird — Messaging System Bug List

---

## 🔴 Critical

**1. Messages not persisted**
`messages` array and `nextMessageId` are not included in `saveData()` / `loadData()`. All messages and read state are lost on page refresh.
- Fix: add `messages` and `nextMessageId` to the save/load data object alongside `lessons`, `swapRequests`, etc.

**2. Crash if conversation has no messages**
In `getConversations()`, `lastMsg` is set to `convMessages[convMessages.length - 1]`. If `convMessages` is empty, `lastMsg` is `undefined` and `lastMsg.text` throws, crashing the entire messages view.
- Fix: guard with `if (!lastMsg) return;` or skip that conversation ID.

**3. Crash when sending with no conversation selected**
In `sendTeacherMessage()`, if `currentConversationId` is `null` the function proceeds and calls `selectConversation(null)`, which filters messages against `null` and likely throws.
- Fix: add `if (!currentConversationId) return;` at the top of `sendTeacherMessage()`.

**4. Inverted read logic for students**
`showStudentMessagesView()` calls `markConversationRead(studentName, studentName)`. Inside `markConversationRead`, messages are marked read where `msg.from !== reader`. Since student messages have `from: studentName`, this marks the student's *own outgoing* messages as read instead of the teacher's incoming messages.
- Fix: the second argument should be `'teacher'` (mark teacher's messages as read from the student's perspective), or the function logic needs a `reader` role parameter.

---

## 🟠 High

**5. Apostrophe in student name breaks onclick**
`showMessagesView()` builds sidebar items with:
```js
onclick="selectConversation('" + conv.id + "')"
```
If a student name contains `'` (e.g. `O'Brien`), this produces malformed HTML and a JS syntax error.
- Fix: use a `data-id` attribute and a delegated event listener, or escape the name with `encodeURIComponent` / `escapeHtml`.

**6. Unread badge not shown on login**
`updateMessageBadges()` is only called after sending a message. On initial teacher or student login, existing unread messages won't light up the badge until something is sent.
- Fix: call `updateMessageBadges()` inside `loginAsTeacher()` and `loginAsStudent()` (or inside `showStudentDashboard()` / `goToDashboard()`).

**7. No way for teacher to start a new conversation**
The teacher can only reply to students who have messaged first. There is no "New Message" or "Message a Student" button.
- Fix: add a button that lets the teacher pick a student from `studentProfiles` and opens a blank thread.

---

## 🟡 Medium

**8. Active conversation highlight is fragile**
After the infinite-loop fix, the sidebar active state is updated by checking `el.getAttribute('onclick').includes("'" + conversationId + "'")`. This is brittle — breaks with special characters in names and will silently fail if the onclick format changes.
- Fix: set `data-conv-id` on each `.conversation-item` and match against that instead.

**9. `onkeypress` is deprecated**
The compose textarea uses `onkeypress` to detect Enter. This event is deprecated, doesn't fire consistently on mobile or with IME input methods, and may be removed in future browsers.
- Fix: replace with `onkeydown`.

**10. Focus not restored after sending**
After `sendTeacherMessage()` or `sendStudentMessage()` refreshes the thread, focus is not returned to the textarea. The user has to click back in to type a follow-up.
- Fix: call `document.getElementById('messageInput')?.focus()` at the end of the send function.

**11. No empty state for students with no messages**
If a student has never exchanged messages with the teacher, `showStudentMessagesView()` renders an empty thread area with no explanation or prompt.
- Fix: add an empty state message like "No messages yet. Send your teacher a message below."

---

## 🟢 Low / Nice to Have

**12. No character limit on message input**
Students and teachers can paste arbitrarily large text blocks. No `maxlength` or client-side validation.

**13. No message deletion**
No way for the teacher to delete a conversation or individual message.

**14. No read receipts for students**
Students can't see whether the teacher has read their message.

**15. Demo messages reference students not in `studentProfiles`**
`"Sophia Garcia"` and `"Noah Wilson"` appear in the seed `messages` array but may not have matching entries in `studentProfiles` or `lessons`, which could cause issues if any code tries to look up their profiles from a conversation ID.

