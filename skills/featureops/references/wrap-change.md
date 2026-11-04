# Wrap a change behind a flag

Put new code behind a flag so it can be turned off in production without a redeploy. Which flag to use comes from `evaluate-change.md` and `find-existing-flag.md`; this file covers where the check goes and how it is written.

## The check runs on every request

A flag is only useful if toggling it takes effect at once. Put the check inside the code path that runs per request, per job run or per render, never around code that runs once at startup.

```typescript
// Wrong: the route is registered at startup. Turning the flag on later changes nothing until a restart.
if (unleash.isEnabled('checkout-stripe')) {
  app.post('/api/checkout', checkoutHandler);
}

// Right: the route always exists, and each request checks the flag.
app.post('/api/checkout', (req, res, next) => {
  if (!unleash.isEnabled('checkout-stripe', { userId: req.user.id })) {
    return res.status(404).json({ error: 'Not found' });
  }
  checkoutHandler(req, res, next);
});
```

The same rule applies to anything registered once:

| Registered at startup | Check instead |
| --- | --- |
| Routes, controllers, routers | Inside the handler, or in a middleware that is always mounted |
| Middleware | Inside the middleware, which is always mounted |
| Scheduled jobs, queue consumers | At the start of each run or each message |
| Dependency injection bindings | Inside the method that is called, or in a factory that resolves per call |
| Values read into a constant or config at boot | At the point of use |

A kill switch is an [inverted feature flag](https://docs.getunleash.io/guides/feature-flag-best-practices#strategies-for-large-organizations-implementing-feature-flags): turning it on disables the thing it is named after. While it is off, which is its normal state, the feature runs. That shapes the check:

- **The enabled branch holds the fallback**, and the code after it runs the feature.
- **Name it after what it turns off** (the policy's kill-switch naming, such as `disable-payments-stripe`), so `isEnabled` reads as "is this disabled?" and the inversion is visible at the call site.
- **The SDK's default keeps the feature running.** `isEnabled` returns false for a flag it doesn't know or can't fetch, so the feature runs as normal if Unleash is unreachable.

```typescript
if (unleash.isEnabled('disable-payments-stripe')) {
  return legacyPayments.charge(order);
}
return stripe.charge(order);
```

## Match the codebase

1. Search for existing flag checks (`flag-usage-patterns.md`) and read one whole file that uses them.
2. Copy what it does: the import, the client or wrapper it calls, the method, how it passes context, and the shape of the check (if block, guard clause, hook).
3. If the codebase has no Unleash SDK set up, read the SDK's current setup page in the Unleash docs before writing the setup. Don't write it from memory; imports and options change between SDK versions.

## Write both paths

- **Pass the context** the rollout needs, at least the user id, so a gradual rollout keeps each user on the same side across requests.
- **Design the off path.** When the flag is off, the old behavior runs. If there is no old behavior, return a clear failure, such as the 404 above, not a silent no-op.
- **Keep the check next to the code it protects**, and check once per decision: evaluate the flag once and pass the result down rather than checking it in several layers.
- **Test both paths**: at least one test with the flag on and one with it off.
