## Reflected XSS (Non-Persistent)

### What is it?
The payload is part of the request sent to the server and is immediately reflected back in the server's HTTP response. It is non-persistent, meaning the payload has to be manually triggered per victim and is never permanently stored on the application's backend or database. It operates strictly within a single, immediate HTTP request-response cycle.

### What might it look like?
Most often, reflected XSS occurs via URL query parameters, search inputs, or form submissions. For example, if a website has a search feature that displays the searched term back to the user (e.g., *"You searched for: **apple**"*), an attacker can swap the word "apple" with an executable script.

#### Example Scenario & Code Flow:
1. **The Vulnerable Server Code (e.g., PHP/Backend):**
   The server takes a URL query parameter named `q` and prints it directly into the HTML without sanitisation or encoding:
   ```html
   <h1>Search Results for: <?php echo $_GET['q']; ?></h1>
   ```

2. **The Attacker's Malicious URL:**
   The attacker crafts a link where the `q` parameter contains a payload and sends it to a victim (via phishing, social media, etc.):
   ```text
   https://example.com<script>alert(document.cookie)</script>
   ```

3. **The Reflected Response:**
   When the victim clicks the link, the server parses the query parameter and reflects it directly into the raw HTML response sent to the victim's browser:
   ```html
   <h1>Search Results for: <script>alert(document.cookie)</script></h1>
   ```
   The victim's browser sees the `<script>` tag embedded in the HTML structure, assumes it is legitimate code from `example.com`, and executes it.

### Common Vectors (Entry Points)
*   **URL Query Parameters:** `?search=payload` or `?id=payload`
*   **URL Path Segments:** `https://example.com` (if the application dynamically echoes the current path onto the page)
*   **HTTP Request Headers:** Custom headers, `User-Agent`, or `Referer` fields, if the application logs them or reflects them back into an error page (e.g., *"Your browser 'payload' is not supported"*).

### Core Constraints & Delivery Mechanics
*   **Phishing Dependency:** Because the code isn't saved on the website, an attacker *must* trick the victim into clicking the specifically crafted link or submitting a malicious form themselves. 
*   **One-to-One Impact:** One link equals one compromised session. It does not naturally spread to other users browsing the site normally.
