# Real-Time Collaborative Editing — System Design

## Question
You built real-time features in Fasal Mitra using Socket.io. If you had to implement a real-time collaborative feature in an enterprise React app — say, multiple users editing the same product simultaneously — how would you architect it?

---

## My Answer
We actually had screen locking in our app — if a product page was open on someone else's screen, we locked it for others. We removed it eventually because our B2B user base is small enough that simultaneous editing rarely happens. But if I had to build it, I would use a unique key with every request and an in-memory queue like Redis or Kafka so simultaneous changes are processed in order. Conflicts like one user deleting a field while another adds to it would need to be handled.

## What I Got Right
- Real production story about screen locking — shows practical experience
- Good pragmatic call to remove it for limited B2B users — product thinking
- Redis/Kafka queue for ordering concurrent requests — thinks beyond frontend
- Identified real conflict scenarios

## What I Got Wrong
- Didn't mention OT (Operational Transformation) or CRDTs — the industry standard solutions
- No mention of optimistic UI — show the user their change immediately, reconcile later
- Didn't mention WebSocket rooms — how to scope updates per product ID
- No mention of reconnection handling — what happens when a user drops connection mid-edit?
- No presence indicators — showing who is currently editing which field

---

## Model Answer
I'd architect this in three layers — connection, state sync, and conflict resolution.

**Connection layer:** WebSocket rooms scoped per product ID. When a user opens a product, they join room `product:${id}`. All events in that room go only to users editing that product.

**State sync layer:** Optimistic UI on the frontend — apply changes locally and show them immediately, then broadcast via WebSocket to other users in the room. Add presence indicators showing "Deepak is editing the price field" so users have awareness of who's doing what, reducing conflicts naturally.

**Conflict resolution layer:** For simple cases, last-write-wins with field-level locking is sufficient — if you're editing price, that field is soft-locked for others. For true collaborative editing like Google Docs, the gold standard is CRDTs (Conflict-free Replicated Data Types) — data structures that mathematically guarantee conflict-free merges regardless of order. For our B2B scale, field-level locking with presence indicators is the practical, shippable solution.

**Edge cases to handle:** reconnection logic — on reconnect, refetch full product state and rehydrate; stale update detection using vector clocks or timestamps.

---

## Key Things to Remember

**Three real-time collaboration patterns:**
1. **Screen locking** — simplest. Lock the whole resource. Poor UX.
2. **Field-level locking** — lock individual fields. Better UX. Good for forms.
3. **CRDTs / OT** — true simultaneous editing. Complex. Google Docs level.

**WebSocket rooms pattern:**
```js
// Server
socket.join(`product:${productId}`)
io.to(`product:${productId}`).emit('field-updated', { field, value, userId })

// Client
socket.on('field-updated', ({ field, value, userId }) => {
  if (userId !== currentUser.id) updateField(field, value)
})
```

**Optimistic UI pattern:**
1. User makes change → update local state immediately
2. Send update to server via WebSocket
3. If server confirms → nothing to do
4. If server rejects → rollback local state and show error

**Presence indicators:**
- Show avatars of active users on the page
- Highlight which field each user is currently editing
- Prevents conflicts before they happen — most practical win

**CRDTs vs OT:**
- Both solve the same problem: merging concurrent edits without conflicts
- OT (Operational Transformation) = Google Docs approach, complex server coordination
- CRDTs = newer, decentralized, mathematically proven conflict-free, used in Figma

**Memory trick:** Room → Optimistic → Presence → Conflicts. ROPC. "Real Operations Prevent Chaos."

---

## Things to Improve
- Read about CRDTs at a high level — you don't need to implement one, just explain what problem they solve
- Practice drawing the WebSocket room architecture on a whiteboard
- Always mention presence indicators — product teams love them and they're practical to build
