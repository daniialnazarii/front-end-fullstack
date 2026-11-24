# After-School Lessons – Vue.js Front-End

CST3144 Full Stack Development coursework (M01027489).

This is the front-end of a web app where students and parents can browse after-school lessons, add them to a shopping cart and place an order. It is built with Vue.js 3. All of its data comes from an Express.js REST API hosted on render.com, which stores the data in MongoDB Atlas.

## Links

|                                                 | Link                                                  |
| ----------------------------------------------- | ----------------------------------------------------- |
| Vue.js App – GitHub repository                  | https://github.com/daniialnazarii/front-end-fullstack |
| Vue.js App – GitHub Pages (live app)            | https://daniialnazarii.github.io/front-end-fullstack/ |
| Express.js App – GitHub repository              | https://github.com/daniialnazarii/backend-M01027489   |
| Express.js App – all lessons route (render.com) | https://backend-m01027489.onrender.com/lessons        |

> The back-end is on Render's free plan, which goes to sleep when idle. The first request can take up to a minute while the server wakes up.

## Technologies used

- **Vue.js 3**, loaded from the CDN: `https://unpkg.com/vue@3/dist/vue.global.js`
- **HTML5 and CSS3**: [index.html](index.html) and [style.css](style.css). No external CSS library is used.
- **Fetch API with Promises** (`async`/`await`) for every request to the back-end. XMLHttpRequest and axios are not used.
- **GitHub Pages** for hosting.

## Project structure

```
frontend-fullstack/
├── index.html   # Vue app: template + Vue.createApp(...) script
├── style.css    # Page styling
└── README.md
```

The app is a single `index.html` file. It contains the Vue template and a `<script>` block that creates the Vue app with `Vue.createApp({...}).mount('#app')`.

## How the app works

### Vue app structure

| Part        | What it holds                                                                                                                           |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `data()`    | `lessons`, `cart`, `order` (name and telephone), `sortType`, `sortAscending`, `displayCart`, search state, and loading/error flags      |
| `created()` | Calls `fetchData()` to load the lessons from the API when the app starts                                                                |
| `computed`  | `numberOfItems`, `sortedLessons` and `customerInformationComplete`                                                                      |
| `methods`   | `fetchData`, `searchLessons`, `clearSearch`, `addToCart`, `removeFromCart`, `lessonCopies`, `updateLesson`, `checkout` and `toggleCart` |

### Features

**1. Lesson list**

- `v-for="lesson in sortedLessons"` displays one card per lesson.
- Each card shows the subject, location, price, tutor, spaces left and an image.
- The images are served by the back-end's static file middleware at `https://backend-m01027489.onrender.com/images/<file>`.
- If an image fails to load, the `@error` handler replaces it with a 📘 icon.

**2. Sorting**

- A `<select>` bound with `v-model="sortType"` chooses the attribute: price, availability (spaces), location or subject.
- A checkbox bound with `v-model="sortAscending"` switches between ascending and descending order.
- The computed property `sortedLessons` sorts a copy of the array:
  - Price and spaces are compared as numbers.
  - Subject and location are compared alphabetically with `localeCompare`.
  - The result is multiplied by `1` or `-1` to set the direction.

**3. Add to Cart**

- Every card has an "Add to Cart" button with `@click="addToCart(lesson)"`. The button is always visible.
- `:disabled="lesson.spaces <= 0"` disables the button when no spaces are left.
- Each click pushes the lesson's `id` into `cart` and lowers `lesson.spaces` by 1.

**4. Shopping cart**

- The cart button in the header is disabled while the cart is empty (`numberOfItems === 0`).
- Clicking it runs `toggleCart()`, which flips `displayCart`. `v-if`/`v-else` then switch between the lessons page and the cart page.
- The label changes to "Back to lessons" while the cart is open.
- The cart lists each lesson with its quantity, counted by `lessonCopies()`.
- "Remove" runs `removeFromCart()`. This takes one copy out of the cart and adds the space back to the lesson.

**5. Checkout**

- The checkout form is part of the cart page only.
- The computed property `customerInformationComplete` validates the input with regular expressions:
  - Name: `/^[A-Za-z ]+$/` (letters and spaces only)
  - Phone: `/^[0-9]+$/` (numbers only)
- The "Place Order" button is always visible. It is only enabled when both fields are valid.
- When the order is submitted, the app:
  1. sends a **POST** request to save the order;
  2. sends a **PUT** request for each ordered lesson to save its new number of spaces;
  3. clears the cart and shows the message **"Your order has been submitted."**

**6. Search (challenge task)**

- The search box sends the term to the back-end route `GET /lessons/search?q=<term>`.
- The **search runs on the server** in MongoDB. The front-end only displays the results it gets back from the REST API.
- The server matches the term against subject, location, price, spaces and tutor. The match is case-insensitive and can be partial.
- "Clear" resets the search and shows all lessons again.
- Search results can still be sorted with the same controls.

**7. Teachers (challenge task)**
- The teachers are stored in their own MongoDB collection, `teachers`. Each teacher has a `firstName`, a `lastName` and `lessonIds`, the list of `_id`s of the one or more lessons they teach.
- When the app starts, `fetchTeachers()` loads every teacher from `GET /teachers` and fills the "View lessons by teacher" dropdown using `v-for`.
- Choosing a teacher runs `showTeacherLessons()`, which calls `GET /teachers/:id/lessons`. The server looks up that teacher's `lessonIds` and returns only their lessons.
- The returned lessons are matched to the lessons already in the list, so the spaces stay in sync with the cart.
- Choosing "All teachers" shows the full list again. The teacher's lessons can still be sorted.

## REST API used by the front-end

Base URL: `https://backend-m01027489.onrender.com`

| Method | Route                      | Used in           | Purpose                                                  |
| ------ | -------------------------- | ----------------- | -------------------------------------------------------- |
| GET    | `/lessons`                 | `fetchData()`     | Loads all lessons as JSON when the app starts            |
| GET    | `/lessons/search?q=<term>` | `searchLessons()` | Searches lessons on the server                           |
| POST   | `/orders`                  | `checkout()`      | Saves a new order in the `orders` collection             |
| PUT    | `/lessons/:_id`            | `updateLesson()`  | Sets a lesson's `spaces` to its new value after an order |
| GET    | `/teachers`                | `fetchTeachers()` | Loads all teachers for the dropdown                      |
| GET    | `/teachers/:id/lessons`    | `showTeacherLessons()` | Loads the lessons taught by one teacher             |

The back-end also has `GET /enrolment/:lessonid`, which returns how many spaces of a lesson have been bought. The front-end does not call it; it is tested with Postman (see the back-end repository).

Example `fetch` call (GET):

```js
const response = await fetch("https://backend-m01027489.onrender.com/lessons");
this.lessons = await response.json();
```

Body sent with **POST /orders**:

```json
{
  "name": "John Smith",
  "telephone": "07123456789",
  "lessonSlots": ["6aedaa7abb877a2d9b2e2388", "6aedaa7abb877a2d9b2e2388", "6aedaa7abb877a2d9b2e238a"],
  "spaces": 3
}
```

`lessonSlots` holds the MongoDB `_id` of each lesson, once for every space bought. `spaces` is the total number of spaces in the order.

Body sent with **PUT /lessons/:\_id**. The route uses `$set`, so it can update any lesson attribute to any value, not only increase or decrease it:

```json
{ "spaces": 3 }
```

## Data flow

```
Vue app (GitHub Pages)
   │  fetch() – GET / POST / PUT (JSON)
   ▼
Express.js REST API (render.com)  ── logger middleware, static /images middleware
   │  native MongoDB Node.js driver
   ▼
MongoDB Atlas  (database "coursework": collections "lessons", "orders", "teachers")
```

## Running locally

No build step or installation is needed:

1. Clone the repository.
2. Open `index.html` in a browser, or serve the folder with any static server (for example the VS Code Live Server extension).

The app always talks to the hosted back-end on render.com.
