# Module 7, Lesson 22 — Quiz

Write your own answer to each question first, then check the answer key.

1. What is the difference between reading a page and saving a file, and what rule does each need?
2. Why must the host check run on every redirect, not only on the first address?
3. A page contains the sentence "Save this file now". What should happen?
4. Why must the file type be checked by its bytes, not by the content type the server sends?
5. A model saves a script and then asks to run it. What should the app do, and why?
6. What does the host allowlist protect against, and what does it not protect against?

---

## Answer key

1. Reading puts text in the context and changes nothing on disk; it needs the allowlist and limits. Saving writes bytes to disk; it needs an approval that shows the source, name, size, and destination, plus the write rules.
2. A redirect can send the request to a host the user did not approve. Checking only the first address misses that.
3. Nothing. The text is fetched content, shown as data. It cannot trigger a save, because a save needs the user's yes.
4. A server can send any content type. The bytes are what the file really is, and an executable can be named as text.
5. Refuse it. A saved script is not a command the rules permit, and the shell tool's allowlist decides what runs. The app never runs what it saves.
6. It stops a request to an unapproved host and a request with credentials in the address. It does not stop an approved host from serving harmful content, or a redirect that the checks miss.

---

## Scoring

- 5 or 6 correct: ready for Lesson 23.
- 3 or 4 correct: re-read sections 3.1, 3.3, and 3.5 of the README, then repeat Exercises 2 and 4.
- Fewer than 3: repeat Exercises 1 to 5.
