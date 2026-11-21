# Task 3 (Variant): Campus Lost & Found — Testing with Jest

The Campus Lost & Found app is already complete: an Express + MongoDB API
with JWT login, and a React client where logged-in users report, edit and
delete lost/found items. **Your job is not to change the app — it is to
write automated tests that prove it works.**

## Running it

```
npm run install:all   # installs server/ and client/ (npm workspaces)
npm test              # runs the backend tests, then the frontend tests
```

Run one side only:

```
npm test --workspace server
npm test --workspace client
```

You do **not** need `server/.env` or a MongoDB connection to run the tests.
The backend tests start their own temporary, in-memory MongoDB (the first
run downloads it, so it may take a minute).

## What is testing?

Software testing means writing code that checks your other code behaves the
way it should. Instead of opening the browser and clicking around by hand
every time you change something, you write the checks down once and the
computer runs them in seconds — as often as you like.

A test usually has three steps — **Arrange, Act, Assert**:

1. **Arrange** — set up what you need (a logged-in user, an existing item…).
2. **Act** — do the thing being tested (send a request, click a button…).
3. **Assert** — check the result is what you expected (status `201`, an error
   message is shown…).

Why bother?

- **Catch bugs early** — a broken feature fails a test before a user sees it.
- **Change code with confidence** — if you refactor and all tests still pass,
  you didn't break anything.
- **Documentation** — reading the tests tells you exactly what the app is
  supposed to do, including the edge cases (`401`, `403`, `409`…).

Common kinds of tests:

| Kind             | Tests…                                                | In this task                                                 |
| ---------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| Unit test        | one small piece (a function, a component) in isolation | the `ItemForm` component, with the API faked               |
| Integration test | several pieces working together                        | real HTTP requests through Express → controllers → MongoDB |
| End-to-end test  | the whole app in a real browser                        | not in this task                                             |

## What is Jest?

[Jest](https://jestjs.io/) is a JavaScript testing framework. It finds your
test files (`*.test.js` / `*.test.jsx`), runs every test in them, and prints
which passed and which failed.

```js
describe('a group of related tests', () => {
  test('one thing that should be true', () => {
    expect(1 + 1).toBe(2)          // passes
  })
})
```

- `describe(name, fn)` — groups related tests.
- `test(name, fn)` — one test. It **passes** if it finishes without a failed
  `expect`, and **fails** otherwise. Use `async` when you need `await`.
- `expect(value)` + a **matcher** — the assertion:
  `toBe` (exact value), `toEqual` (same contents for objects/arrays),
  `toHaveLength`, `toHaveProperty`, `toHaveBeenCalledWith`, and `.not`
  to flip any of them (`expect(x).not.toBe(5)`).
- `beforeEach` / `afterEach` / `beforeAll` / `afterAll` — code that runs
  around your tests (e.g. reset mocks, empty the database).
- `test.todo('...')` — a placeholder for a test that isn't written yet.
  Jest lists it as *todo*.
- **Mocks** — `jest.fn()` creates a fake function that remembers how it was
  called and returns whatever you tell it to. `jest.mock('../api')` swaps a
  whole module for fakes, so the frontend tests never call the real server.

Jest is helped by two libraries here:

- **[Supertest](https://github.com/ladjs/supertest)** (backend) — sends HTTP
  requests straight to the Express `app` without starting a server:
  ```js
  const res = await request(app).post('/api/items').set('Authorization', `Bearer ${token}`).send({ title: 'Phone' })
  expect(res.status).toBe(201)
  ```
- **[React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)**
  (frontend) — renders a component into a fake browser page (jsdom) and lets
  you find elements the way a user would (`screen.getByRole`,
  `getByPlaceholderText`, `findByText`). Its companion **user-event** types
  and clicks (`await userEvent.type(input, 'Phone')`,
  `await userEvent.click(button)`), and **jest-dom** adds matchers such as
  `toBeInTheDocument()` and `toHaveValue()`.

`getBy…` finds an element that is there right now, `findBy…` *waits* for it
(use it after anything async, e.g. after a request resolves), and
`queryBy…` returns `null` instead of throwing — use it to assert something
is **not** there.

## Your task

You edit **exactly two files**. Everything else (Jest config, the in-memory
database, mocks, helpers) is already set up — do not change the app code.

Each file contains one finished example test and **two** `test.todo(...)`
lines. Turn each `test.todo` into a real `test` that passes — **4 tests in
total**. When you're done, `npm test` should report **0 todo** and
**0 failed**.

### File 1 — backend: `server/tests/items.test.js`

Integration tests for `/api/items`, using Supertest. Helpers provided:

- `registerAndGetToken(name, email)` → a JWT for a new user (call it twice
  with different emails to get two users).
- `createItem(token, overrides)` → creates an item as that user and returns
  it (default: `Black backpack`, `lost`, `accessories`, at `Library`).

Write these two tests:

1. **Duplicate report → `409`.** Reporting the same `title` at the same
   `location` twice is rejected: the second `POST /api/items` returns `409`.
2. **Not the reporter → `403`.** Only the person who reported an item can
   edit it. Create an item as one user, then `PATCH /api/items/:id` with a
   second user's token — it returns `403`.

### File 2 — frontend: `client/src/pages/ItemForm.test.jsx`

Component tests for the Report Item page
(`client/src/pages/ItemForm.jsx`), using React Testing Library. The `api`
module is mocked, so no server is needed. Helper provided:

- `renderAt('/items/new')` — renders the page. After a successful save the
  page navigates to `/items`, which shows the text `Items page` — check for
  it to prove the navigation happened.

Write these two tests:

1. **Submitting the form.** Fill in the fields and click Save. Check that
   `api.post` was called with `'/items'` and exactly
   `{ title, status, category, location, description }` (no `reportedBy`),
   and that the page navigated to `/items`.
2. **Showing the server's error.** Make `api.post` fail with
   `{ response: { data: { message: 'Duplicate item at this location' } } }`,
   submit, and check that message appears on the page and you did **not**
   navigate away.

### Checking your tests actually test something

A test that always passes is useless. For each test, break the app on
purpose for a moment — e.g. delete the ownership check in
`server/src/controllers/itemController.js`, or comment out the
`setError(...)` inside the `catch` of `onSubmit` in `ItemForm.jsx` — and make sure the right test **fails**. Then
undo the change.

You're expected to use AI tools while writing these. But you should be able
to explain, for any test, *what* it checks, *why* it would fail if the
feature broke, and what every line does. We will ask.
