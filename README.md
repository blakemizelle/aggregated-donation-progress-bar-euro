
Okay, absolutely. It's crucial to document this clearly, both for maintenance and for the end-user. Here's a breakdown:

## Customer Instructions: Using the Combined Campaign Progress Bar

This feature allows you to link multiple donation pages together into a "campaign". Any page included in the campaign will display a progress bar showing the *total amount raised across all pages in that campaign*, calculated against the goal set on the page currently being viewed.

**How to Set Up a Fundraising Campaign:**

1.  **Choose a Campaign Identifier:**
    *   Think of a short, unique name for your campaign (e.g., `end_of_year_2024`, `building_fund`, `rapid_response`).
    *   Use only lowercase letters, numbers, and underscores (`_`). Avoid spaces or special characters other than the underscore.

2.  **Identify Campaign Pages:**
    *   List all the Donation (v2) pages that should contribute to and display this campaign's total.

3.  **Tag Your Pages:**
    *   Go to your NationBuilder **Control Panel**.
    *   Navigate to **Website** > Select your site > **Pages**.
    *   Find the first donation page for your campaign and click to edit it.
    *   Go to the **Settings** tab, then the **Basics** sub-tab.
    *   In the **Tags** field, add a tag following this **exact format**:
        `aggregated_campaign | YourCampaignIdentifier`
        *   Replace `YourCampaignIdentifier` with the name you chose in step 1. Make sure to include `aggregated_campaign`, a space, the pipe symbol (`|`), another space, and then your identifier.
        *   *Example:* `aggregated_campaign | end_of_year_2024`
    *   **CRITICAL:** Add the **exact same tag** to *every single* donation page that should be part of this campaign. The aggregation only works if the tags match perfectly.
    *   Save the settings for each page.

4.  **Set Fundraising Goals:**
    *   The progress bar shows the percentage raised towards the goal set on the *specific page the supporter is viewing*.
    *   For each page in the campaign, go to **Settings** > **Goal**.
    *   Ensure you have set an **Amount** goal (e.g., €10000). **The percentage calculation will not work correctly if a page has no goal or a goal of 0.**
    *   *Recommendation:* For consistency, consider setting the same overall campaign goal amount on every page participating in the campaign.

**How it Works:**

*   Any donation page tagged with `aggregated_campaign | YourCampaignIdentifier` will automatically switch from showing its individual progress to showing the combined total for *all* pages sharing that *exact* tag.
*   The text will show the combined total raised across the campaign.
*   The progress bar's percentage reflects the combined total raised divided by the goal set on the *current page*.

**Managing Campaigns:**

*   **Multiple Campaigns:** You can run several independent campaigns simultaneously by using different `YourCampaignIdentifier` values in the tags (e.g., `aggregated_campaign | campaignA`, `aggregated_campaign | campaignB`). Pages only aggregate with others sharing their specific campaign tag.
*   **Removing a Page:** To stop a page from contributing to or displaying a campaign total, simply edit the page (**Settings > Basics**) and remove the relevant `aggregated_campaign | ...` tag. It will revert to showing its individual progress bar.
*   **Updates:** Donation totals are updated based on NationBuilder's caching; allow a few minutes for new donations to be reflected in the combined total.

---

## Technical Documentation: Combined Campaign Progress Bar Logic

**Goal:** To display a progress bar on specific Donation (v2) pages that aggregates the total amount raised from all pages sharing the same designated "campaign" tag.

**Files Modified:**

*   `pages_show_donation_v2_wide.html`: Contains the primary logic for detection and calculation.
*   `_combined_progress.html`: Handles the final display formatting and rendering of the paragraph and progress bar elements.

**Logic Flow in `pages_show_donation_v2_wide.html`:**

1.  **Campaign Detection:**
    *   Initializes `is_campaign_page = false`.
    *   Loops through the current `page.tags`.
    *   For each tag, it checks if its name (accessed via `tag.name`) `contains` the string `"aggregated_campaign | "`.
    *   If a match is found, it sets `is_campaign_page = true`, captures the full matching tag name into `campaign_tag_name`, and breaks the loop (processing only the first matching campaign tag found).

2.  **Conditional Execution (`{% if is_campaign_page %}`):**
    *   If `is_campaign_page` is true, the combined logic executes. Otherwise, the standard progress bar logic in the `{% else %}` block runs.

3.  **Calculate Current Page Contribution:**
    *   Calculates the donation amount for the *current* page in cents (`display_page_cents`) by processing `page.donations_amount_format` (removing currency symbols/separators, converting to cents).

4.  **Fetch Campaign Pages:**
    *   Uses the captured `campaign_tag_name` from the initial detection loop.
    *   **Immediately** fetches the associated pages using `tag.alphabetical_pages_no_pagination` (called directly on the `tag` object within the detection loop due to observed Liquid scoping behavior). Stores the result in `pages_for_this_tag`.

5.  **Aggregate Source Totals:**
    *   Initializes `campaign_total_cents = 0` (before the detection loop).
    *   Loops through the fetched `pages_for_this_tag`.
    *   For each `loop_page`, it converts `loop_page.donations_amount_format` into cents (`loop_cents`) using string filters (`remove`, `replace`, `times`, `round`).
    *   It adds `loop_cents` to `campaign_total_cents`. (Note: Based on testing, `tag.alphabetical_pages_no_pagination` might exclude the current page itself, which is handled next).

6.  **Combine Totals:**
    *   *After* the tag detection and source page aggregation loop, it adds the current page's contribution (`display_page_cents`) to the aggregated `campaign_total_cents`.

7.  **Calculate Goal Cents:**
    *   Retrieves the formatted goal string from `page.donation_v2.amount_goal_format`.
    *   Converts this string into cents (`current_page_goal_cents`) using filters (`remove`, `replace`, `times`, `round`). Uses `default` filters for safety.

8.  **Calculate Percentage:**
    *   Calculates `campaign_percentage` if `current_page_goal_cents > 0`.
    *   Crucially uses `| times: 1.0 | divided_by:` to ensure floating-point division.
    *   Caps the percentage at 100.

9.  **Prepare Data for Partial:**
    *   Assigns the formatted goal string (`page.donation_v2.amount_goal_format` or defaults) to `goal_format`.
    *   Prepares the parameters for the include:
        *   `percent`: The calculated `campaign_percentage`.
        *   `amount_in_cents`: The final `campaign_total_cents` (integer).
        *   `goal_cents_raw`: The calculated `current_page_goal_cents` (integer).
        *   `goal`: The formatted `goal_format` string (for display).

10. **Include Display Partial:**
    *   Calls `{% include "combined_progress" ... %}` passing the calculated/formatted values.

**Logic Flow in `_combined_progress.html`:**

1.  **Receive Variables:** Accepts `percent`, `amount_in_cents`, `goal_cents_raw`, and `goal` from the `include` tag. Uses `default` filters for safety.
2.  **Format Amount:**
    *   Takes `received_cents` (from `amount_in_cents`).
    *   Performs manual formatting to create a Euro string (`display_amount`):
        *   Divides by `100.0`.
        *   Rounds to 2 decimal places.
        *   Replaces `.` with `,`.
        *   Prepends `€`.
        *   Ensures two decimal places are visually present (adds `,00` or `0`).
    *   *Note:* This manual formatting avoids the unreliable `number_with_delimiter` filter but does *not* include thousands separators.
3.  **Render Paragraph:** Outputs the paragraph text directly, embedding the manually formatted `display_amount` and the received formatted `display_goal` string.
4.  **Render Progress Bar:**
    *   Uses `display_percent` for `data-percent`, `style="width:..."`, and `aria-valuenow`.
    *   Uses `received_goal_cents` (raw integer) for `data-goal`.
    *   Uses `received_cents` (raw integer) for `data-amount`.

This structure ensures calculations happen centrally, formatting happens display-time in the partial, and relies on the most robust Liquid methods identified through debugging.
