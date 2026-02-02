# From Junior to Mid-Level: Structuring an Express Backend

## Background

When starting with Express / MERN backends, it is very common to put all logic inside controllers.

This works well for small projects, but as the application grows, controllers become hard to maintain and risky to change.

This repository documents what I learned about moving from a controller-heavy design to a service-based design, and why this shift matters when going from a junior to a mid-level backend engineer.

## 🧠 Why This Structure Exists

### Junior-Level Thinking ❌
- Controllers talk directly to MongoDB
- Business logic mixed with request handling
- Same validation repeated everywhere
- Hard to test and hard to scale

### Mid-Level Thinking ✅
- Controllers only handle HTTP (req/res)
- Services contain business logic
- Models only define schemas
- Utils handle shared logic
- Each layer has **one responsibility**

## The Junior Pattern: Fat Controllers

In many beginner Express projects, controllers are responsible for:

- Reading HTTP requests
- Validating input
- Querying the database
- Applying business rules
- Performing side effects (logs, files, etc.)

Example:
```
// controllers/deviceController.js
exports.verifyDevice = async (req, res) => {
  const { macId } = req.body;

  if (!macId) {
    return res.status(400).send("MAC ID required");
  }

  const device = await BT.findOne({ macId });

  if (!device) {
    return res.status(401).send("Invalid device");
  }

  if (device.blocked) {
    return res.status(403).send("Device blocked");
  }

  res.send({ success: true });
};
```

## Why This Becomes a Problem

This approach works, but it has hidden issues:

- Controllers become fragile and hard to modify
- Business logic cannot be reused outside HTTP routes
- Heavy operations (DB, file I/O) mix with critical paths like login
- A failure in logging or file handling can break authentication

As the application grows, changing one thing often breaks another.

## The Mid-Level Pattern: Thin Controllers + Services

A cleaner approach is to move business logic into services.
Controllers only translate HTTP requests and responses.

```
// services/deviceService.js
async function verifyDevice(macId) {
  if (!macId) {
    throw new Error("MAC ID missing");
  }

  const device = await BT.findOne({ macId });

  if (!device) {
    return { ok: false, reason: "NOT_FOUND" };
  }

  if (device.blocked) {
    return { ok: false, reason: "BLOCKED" };
  }

  return { ok: true };
}

module.exports = { verifyDevice };
```
```
// controllers/deviceController.js
const deviceService = require("../services/deviceService");

exports.verifyDevice = async (req, res) => {
  try {
    const result = await deviceService.verifyDevice(req.body.macId);
    res.send(result);
  } catch (err) {
    res.status(400).send(err.message);
  }
};
```
## Mental Model Shift

Junior thinking:
"Where should I write this code?"

Mid-level thinking:
"What responsibility does this code have?"

Controllers speak HTTP. Services make decisions.

Once business logic lives in services, it can be reused, tested, and evolved without touching request-handling code.
