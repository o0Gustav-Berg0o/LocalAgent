# Module 7, Lesson 22 — Review Checklist

## Reading

- [ ] The host check parses the address and checks the parsed host, with normalisation.
- [ ] Only https is allowed, and credentials in an address are refused.
- [ ] Redirects are followed by the application, one at a time, and each new host is checked.
- [ ] The timeout and the body size limit are enforced while reading.

## Content

- [ ] Hidden elements and scripts are removed from the text.
- [ ] The text is labelled as untrusted data in the tool result.
- [ ] A page cannot cause a save or a rule change by its text.

## Saving

- [ ] A save needs a yes at a consent screen that shows the source, name, size, and destination.
- [ ] The consent screen is the same code as the Module 1 download screen.
- [ ] The save writes to a temporary file, checks the size, and renames.
- [ ] A name that exists is refused, not overwritten.

## Type

- [ ] The file's first bytes are checked against its extension.
- [ ] An executable signature and an executable extension are refused.

## Execution

- [ ] The app never runs a file it saved. Tested with a script that would create a marker file.

## Limits

- [ ] Fetch calls per question are limited, and refused attempts are logged without bodies.

## Reflection

- [ ] I tested the redirect gap on purpose and recorded the result.
- [ ] I can say what the allowlist does not protect against.
