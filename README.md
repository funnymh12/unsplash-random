# Markdown Unsplash Image Updater (GitHub Actions)

This project uses a GitHub Action to automatically fetch a random image from Unsplash daily and update a specified Markdown file with the new image URL. This provides a stable image URL in your Markdown for better loading, while still refreshing the image content regularly.

## Featured Image

<!-- UNSPLASH_IMAGE_START -->
![nature](https://images.unsplash.com/photo-1518837695005-2083093ee35b?ixid=M3w1MzY4Njd8MHwxfHJhbmRvbXx8fHx8fHx8fDE2OTk4NjU0NTR8&ixlib=rb-4.0.3&w=2560) <!-- Replace this line with your initial image or leave it for the action to populate -->
<!-- UNSPLASH_IMAGE_END -->

## How it Works

1.  **GitHub Action:** A workflow defined in `.github/workflows/update-unsplash-image.yml` runs on a daily schedule (or can be triggered manually).
2.  **Fetch Image:** The action calls the Unsplash API to get a random image based on specified criteria (e.g., query 'nature', landscape orientation).
3.  **Update Markdown:** The action then finds placeholder comments (`<!-- UNSPLASH_IMAGE_START -->` and `<!-- UNSPLASH_IMAGE_END -->`) in the target Markdown file (e.g., this `README.md`).
4.  **Commit Changes:** The content between these placeholders is replaced with the new Unsplash image URL, and the changes are committed and pushed back to the repository.

## Setup

1.  **Unsplash API Key:**
    *   Go to [unsplash.com/developers](https://unsplash.com/developers) and create an account or log in.
    *   Create a new application. You'll get an **Access Key**. The free tier is sufficient for this purpose.

2.  **GitHub Repository Secret:**
    *   In your GitHub repository, go to `Settings` > `Secrets and variables` > `Actions`.
    *   Click `New repository secret`.
    *   Name the secret `UNSPLASH_API_KEY`.
    *   Paste your Unsplash Access Key as the value for the secret.

3.  **Configure the Workflow (Optional):**
    Open `.github/workflows/update-unsplash-image.yml`.
    You can customize the following environment variables within the `Fetch Unsplash Image and Update Markdown` step:
    *   `MARKDOWN_FILE`: The path to the Markdown file you want to update (default: `README.md`).
    *   `UNSPLASH_QUERY`: The search term for Unsplash images (default: `nature`).
    *   `UNSPLASH_ORIENTATION`: The desired image orientation (default: `landscape`). Can be `landscape`, `portrait`, or `squarish`.

4.  **Add Placeholders to Your Markdown File:**
    In the Markdown file you want to update (e.g., `README.md`, or another file specified in `MARKDOWN_FILE`), add the following placeholders where you want the image to appear:

    ```markdown
    <!-- UNSPLASH_IMAGE_START -->
    <!-- This content will be replaced by the GitHub Action -->
    ![Placeholder Image Text](https://example.com/placeholder.jpg)
    <!-- UNSPLASH_IMAGE_END -->
    ```
    The action will replace everything between these two comment lines (inclusive of the comments themselves if the script is modified, but by default it replaces the content *between* them).

5.  **Enable GitHub Actions:**
    Ensure GitHub Actions are enabled for your repository (they usually are by default).

## How the Original Cloudflare Worker Method Works (Alternative)

*(The following describes the initial Cloudflare Worker approach. The current setup uses GitHub Actions as described above.)*

The Cloudflare Worker (`index.js` and `wrangler.toml`) listens for requests on specific URL paths and redirects them to the corresponding Unsplash Source URL. This means you get a fresh, high-quality image from Unsplash each time the Markdown is rendered (if the image isn't cached by the browser or CDN).

### Worker Usage (If you choose to use it instead of/alongside Actions)

Once deployed, you can use the following URL structures in your Markdown:

**1. Random Image:**
   - `https://<your-worker-url>/random`
   - `https://<your-worker-url>/random/WIDTHxHEIGHT`

**2. Image by Keyword:**
   - `https://<your-worker-url>/keyword`
   - `https://<your-worker-url>/keyword/WIDTHxHEIGHT`
   - `https://<your-worker-url>/keyword/WIDTH/HEIGHT`

(And other paths as defined in `index.js`...)

Replace `<your-worker-url>` with the actual URL of your deployed Cloudflare Worker.

### Worker Deployment

1.  **Install Wrangler CLI:** `npm install -g @cloudflare/wrangler`
2.  **Login:** `wrangler login`
3.  **Configure `wrangler.toml`:** Update `name`, `compatibility_date`, and `account_id`.
4.  **Publish:** `wrangler publish`

## Choosing Between Methods

*   **GitHub Actions (Recommended for this setup):**
    *   **Pros:** Image URL in Markdown is static, leading to more reliable embedding and caching by browsers/CDNs. No external service dependency at render time (once updated).
    *   **Cons:** Image only updates when the Action runs (e.g., daily).
*   **Cloudflare Worker:**
    *   **Pros:** Image can be different on every page load (if not cached), providing more dynamic content. Simpler initial Markdown link.
    *   **Cons:** Relies on the Worker and Unsplash Source being available at render time. Can lead to broken images if the worker is down or Unsplash Source has issues. Might be slower if images are not cached effectively.

Feel free to remove the Cloudflare Worker files (`index.js`, `wrangler.toml`) if you solely prefer the GitHub Actions method.