# NationBuilder Combined Donation Progress Bar

## Description

This project implements a custom feature for NationBuilder themes that allows specific Donation (v2) pages to display an aggregated fundraising progress bar. Instead of showing only its own progress, a participating page will display the combined total amount raised across *all* pages belonging to the same designated "campaign", measured against the goal set on the page currently being viewed.

**NOTE:** This code was developed and tested using the official NationBuilder **Momentum** theme ([https://momentum-theme.nationbuilder.com](https://momentum-theme.nationbuilder.com)). While the core Liquid logic should be adaptable, the specific HTML template file (`pages_show_donation_v2_wide.html`) and CSS class names used for the progress bar (`progress`, `progress-bar`) are based on Momentum. Adaptations may be necessary for other themes.

## Problem Solved

Standard NationBuilder donation pages show progress only towards their individual goals. Organizations running coordinated campaigns across multiple donation pages (e.g., for different appeals, A/B tests, or team pages under one umbrella) often want to display the *overall* campaign progress on each participating page to create a sense of collective effort and momentum. This code provides a Liquid-based solution to achieve that within the theme layer.

## Solution Overview

The solution modifies the main Donation (v2) page template (`pages_show_donation_v2_wide.html` or equivalent) and uses a dedicated partial (`_combined_progress.html`) for display.

1.  **Campaign Detection:** The code checks if the currently viewed page has a tag matching the pattern `aggregated_campaign | CampaignIdentifier`.
2.  **Conditional Logic:** If such a tag is found, the combined progress bar logic runs; otherwise, the page displays its standard progress bar.
3.  **Page Fetching:** When displaying a combined bar, the code identifies the specific campaign tag object and uses the `tag.alphabetical_pages_no_pagination` Liquid method to fetch up to 500 pages associated with that exact tag. (This happens immediately within the tag detection loop due to Liquid scoping behavior).
4.  **Aggregation:** It loops through the fetched pages, extracts their donation totals (`donations_amount_format`), converts these currency strings to cents, and sums them. It also adds the current page's own donation total (in cents) to this sum.
5.  **Goal & Percentage Calculation:** It retrieves the goal amount for the *current* page by converting the formatted goal string (`page.donation_v2.amount_goal_format`) to cents. It then calculates the percentage of the total aggregated cents raised against this goal.
6.  **Display:** The final calculated percentage, the raw aggregated total (in cents), and the raw goal (in cents) are passed to the `_combined_progress.html` partial.
7.  **Formatting & Rendering (`_combined_progress.html`):** The partial formats the aggregated cents total into a display-friendly Euro string (e.g., €312,00 - *note: no thousands separator*) and renders the paragraph text and the progress bar HTML structure, populating the necessary `data-*` attributes and style widths.

## Features

*   Aggregates donation totals from multiple specified Donation (v2) pages.
*   Displays a combined progress bar and total raised text on participating pages.
*   Uses the goal set on the *currently viewed page* for percentage calculation.
*   Conditionally displays either the combined bar or the standard individual bar based on tagging.
*   Supports multiple independent campaigns via different tag identifiers.
*   Implemented purely using NationBuilder Liquid templating (no external services required).

## Prerequisites

*   NationBuilder account with theme access.
*   Website using the **Momentum theme** or a theme with a similar structure for Donation (v2) pages and progress bars.
*   Using Donation (v2) page types for the campaign pages.

## Installation / Setup

There are two primary ways to implement this feature:

**Option 1: Site-Wide Implementation (Overwrites Theme Default)**

*   **Risk:** High. This replaces your theme's default Donation (v2) template.
*   **Benefit:** All Donation (v2) pages on your site can potentially use the combined progress bar feature simply by adding the appropriate tags.

1.  **Backup Your Theme:** Crucial first step. Create a backup or duplicate of your current theme.
2.  **Identify Donation Template:** Determine the main Donation (v2) template file used by your theme (commonly `pages_show_donation_v2_wide.html` for themes like Momentum).
3.  **Overwrite Template:** Replace the contents of your theme's default Donation (v2) template with the code from the provided `pages_show_donation_v2_wide.html` file. **Warning:** This will overwrite any existing customizations in that file. Carefully merge changes if necessary.
4.  **Add Partial:** Copy the provided `_combined_progress.html` file into the root of your theme's template directory.
5.  **Upload Changes:** Upload the modified template and the new partial file to your NationBuilder theme.

**Option 2: Per-Page Implementation (Uses Custom Templates)**

*   **Risk:** Low. Does not affect the theme's default donation page behavior.
*   **Benefit:** Allows you to enable the combined progress bar only on specific donation pages you choose.

1.  **Backup Your Theme:** Always recommended.
2.  **Add Partial:** Copy the provided `_combined_progress.html` file into the root of your theme's template directory.
3.  **For each Donation Page where you want the combined bar:**
    *   Navigate to the specific Donation (v2) page in your control panel (**Website** > Select site > **Pages**).
    *   Click **Template**.
    *   Paste the *entire contents* of the provided `pages_show_donation_v2_wide.html` file into this page-level template editor, replacing any existing code.
    *   Click **Save and publish changes to this template**.
4.  **Note:** Pages using this custom template will aggregate totals from *all* pages tagged with their specific campaign tag, even pages still using the default theme template.

## Configuration & Usage

Follow these steps to configure and use the combined progress bar:

1.  **Choose a Campaign Identifier:**
    *   Select a short, unique name for your campaign (e.g., `end_of_year_2024`, `building_fund`). Use lowercase letters, numbers, and underscores (`_`).
2.  **Identify Campaign Pages:**
    *   Determine which Donation (v2) pages belong to this campaign.
3.  **Tag Your Pages:**
    *   Go to the **Settings > Basics** tab for each participating donation page.
    *   Add a tag in the **exact format**: `aggregated_campaign | YourCampaignIdentifier`
        *   *(Example: `aggregated_campaign | end_of_year_2024`)*
    *   **Apply the *identical* tag to ALL pages in the campaign.**
4.  **Set Fundraising Goals:**
    *   For each page in the campaign, go to **Settings > Goal**.
    *   Set a non-zero **Amount** goal (e.g., €10000). **The percentage calculation requires a valid goal on the page being viewed.**
    *   Consider setting the same goal amount on all pages in the campaign for visual consistency.
5.  **Clear Caches:** Clear your NationBuilder theme cache (**Website > Theme > Clear theme cache**) and your browser cache after setup or changes.

## Technical Details

*   **Base Theme:** Assumes structure from NationBuilder **Momentum** theme.
*   **Tag Detection:** Uses `tag.name contains "aggregated_campaign | "` within a loop over `page.tags`.
*   **Page Fetching:** Relies on `tag.alphabetical_pages_no_pagination` called immediately on the tag object within the detection loop. This returns up to 500 pages.
*   **Currency Conversion:** Uses Liquid string filters (`remove`, `replace`) and math filters (`times`, `round`, `plus`) to convert formatted Euro strings (e.g., `€1.234,56`) to integer cents for calculation.
*   **Goal Conversion:** Uses Liquid string/math filters to convert the formatted goal string (`page.donation_v2.amount_goal_format`) into integer cents.
*   **Percentage Calculation:** Uses `| times: 1.0 | divided_by:` to ensure float division.
*   **Display Formatting:** The `_combined_progress.html` partial handles formatting the final total cents into a display string (e.g., `€1234,56`), specifically avoiding the `number_with_delimiter` filter due to observed errors.
*   **Progress Bar Data:** The `div.progress` element uses `data-percent`, `data-goal` (in cents), and `data-amount` (in cents) attributes, consistent with standard NationBuilder progress bars.

## Limitations

*   **Theme Dependency:** Designed for the NationBuilder **Momentum** theme. Adaptation required for other themes, especially regarding template names and CSS classes for the progress bar.
*   **Currency Formatting:** The current manual formatting in `_combined_progress.html` does **not** include thousands separators (e.g., it displays `€12345,67` instead of `€12.345,67`). Modifying this requires more complex Liquid logic or potentially JavaScript.
*   **Page Limit:** `tag.alphabetical_pages_no_pagination` returns a maximum of 500 pages. If a single campaign tag is applied to more than 500 pages, the aggregation will be incomplete.
*   **Performance:** While efficient for typical use cases, performance *could* degrade slightly if a very large number of pages (~hundreds) share the same campaign tag, due to the processing loop.
*   **Goal Requirement:** Relies on a valid, non-zero Amount goal being set on *each* page displaying the combined bar for correct percentage calculation.

## Troubleshooting

*   **Bar Not Showing/Incorrect Total:**
    *   Verify the **exact** same `aggregated_campaign | CampaignIdentifier` tag is present on *all* intended pages. Check for typos, extra spaces, or case differences.
    *   Ensure all participating pages are Donation (v2) page types.
    *   Clear NationBuilder theme cache and browser cache thoroughly.
*   **Percentage Incorrect / Stuck at 0%:**
    *   Verify that the page you are viewing has a non-zero **Amount** goal set under **Settings > Goal**.
*   **Liquid Errors:** Ensure you copied the code exactly. Errors like "wrong number of arguments" often relate to currency/number formatting filters; the current version avoids known problematic filters.
