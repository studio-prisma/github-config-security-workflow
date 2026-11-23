## What changes

<!-- One or two sentences. What does this PR do, and why? -->

## Type

- [ ] Scanner version bump
- [ ] Behaviour change (what blocks, what reports)
- [ ] Documentation
- [ ] CI / tooling

## Security impact

- [ ] Changes how a scanner binary is downloaded or verified
- [ ] Changes what makes the gate fail
- [ ] Adds an input that ends up in a shell step
- [ ] No security-relevant surface touched

If any box above is checked, describe the threat model change:

<!-- ... -->

## Verification

- [ ] `actionlint` passes locally
- [ ] New scanner versions are at least two weeks old
- [ ] Self-test (`validate.yml`) is green
- [ ] No hardcoded hostnames, credentials, or absolute paths added

## Related issues

Closes #
