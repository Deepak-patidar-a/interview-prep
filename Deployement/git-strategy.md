# Git Strategy — Multi-Customer Branch Management

## Question
When you fix a critical bug in base code, how do you make sure that fix reaches all customers without overwriting their custom enhancements? What problems have you faced?

---

## My Answer
We have a base code branch called develop. When deploying, we create a release branch (e.g. cust1_release) where we first merge develop (base code) and then merge cust1_main (customer specific code) on top. This way critical fixes reach every customer without overriding custom code. Main problem is merge conflicts and overriding issues if the correct order is not followed.

## What I Got Right
- Correct release branch strategy — merging develop first then customer branch on top is the right order
- Honestly mentioned merge conflicts as a real problem
- Understood why the merge order matters

## What Could Be Added
- Did not mention who is responsible for resolving conflicts
- Did not mention conventions to reduce conflicts
- Did not address the scaling limitation of this approach

---

## Model Answer
We use a release branch strategy — for each deployment we create a customer release branch, merge develop first (base code), then merge the customer branch on top. This ensures all base code fixes including security patches reach every customer while keeping their customizations intact. The developer making the base code change is responsible for resolving any conflicts. To reduce conflicts we follow a convention of keeping customer-specific code in separate files and configs so it does not overlap with base code files. The main limitation of this approach is it does not scale well beyond a handful of customers — the long-term solution would be a feature flag system where all customers run the same code and behavior is controlled through configuration.

---

## Key Things to Remember

**Release branch merge order — this is critical:**
1. Create `cust1_release` branch
2. Merge `develop` into it first (base code, bug fixes)
3. Merge `cust1_main` on top (customer specific code wins conflicts)
4. Test and deploy `cust1_release`

**Why order matters:**
- If you merge customer branch first then develop on top, base code can override customer customizations
- Customer branch must always be merged last so it takes priority in conflicts

**Reducing merge conflicts — conventions:**
- Customer-specific code should never modify the same files as base code
- Keep customizations in separate config files, override files, or separate components
- Sync customer branches from develop frequently — small merges are better than one big merge
- The longer branches diverge, the worse the conflicts become

**The pattern name:**
- GitFlow with tenant overlays

**Scaling limitation:**
- Branch-per-customer works at 3 customers
- At 30 customers it becomes unmanageable — too many branches, too many pipelines, too many conflict resolutions
- Solution at scale — feature flags
  - One branch, one codebase
  - Customer behavior controlled through database configuration
  - Toggle features on/off per customer without code changes

**Feature flag concept:**
```javascript
// Instead of separate code per customer
if (featureFlags.isEnabled('new-dashboard', customerId)) {
  return <NewDashboard />
}
return <OldDashboard />
```

---

## Things to Improve
- Research feature flag tools — LaunchDarkly, Unleash, or custom implementation
- Ask team: what conventions exist to reduce customer branch conflicts?
- Understand git cherry-pick as an alternative to merging for single bug fixes
