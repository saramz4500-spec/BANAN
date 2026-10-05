# Banan Interactive Survey

An interactive research survey for **Banan**, a project focused on helping hearing people learn Saudi Sign Language. The survey turns a familiar questionnaire into a small retro desktop experience, with playful windows, tabs, original icons, and simple sound effects.

The interface is in Arabic with right-to-left layout, and each survey question includes an English translation. It uses plain HTML, CSS, and JavaScript, with Google Apps Script handling submissions to Google Sheets.

## The Participant Experience

Participants open the “اضغط هنا” desktop file, enter a name or nickname, age, and gender, then work through three sections. Earlier sections remain available for review, and answers stay in place when moving between tabs.

The survey includes required-field checks and conditional questions. Participants who identify as Deaf or hard of hearing are thanked after the first section, since this survey currently focuses on hearing participants. Age groups are calculated from the age entered at the start; participants must be at least 12.

After a confirmed final save, a separate “إجاباتي” file appears on the desktop. It opens a text summary of the participant’s answers and offers a downloadable UTF-8 text file. The thank-you window also includes options to share the survey link.

## Project Files

| File | Purpose |
| --- | --- |
| `index.html` | Desktop layout and page entry point |
| `style.css` | Retro styling, responsive layout, and animations |
| `app.js` | Survey questions, navigation, validation, sounds, and submissions |
| `Code.gs` | Google Apps Script code for saving and updating responses |
| `README.md` | Setup and testing instructions |

No build step or package installation is needed. Keep the three frontend files in the same directory. For a quick interface check, open `index.html` in a browser; test submissions and sharing on the hosted site before distributing it.

## Connect Google Sheets

The current frontend already contains a deployment URL in `app.js`. To connect your own response sheet:

1. Create a Google Sheet and open **Extensions → Apps Script** from that sheet.
2. Replace the default script with the contents of `Code.gs` and save.
3. Select `setupBanan` and run it once. Authorize the script when prompted. This stores the spreadsheet ID and creates a `Responses` tab with column headers.
4. Choose **Deploy → New deployment → Web app**. Set **Execute as** to your account and **Who has access** to **Anyone**.
5. Deploy and copy the web app URL ending in `/exec`.
6. In `app.js`, replace the existing value of `URL_` with your deployment URL:

```javascript
const URL_ = 'https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec';
```

After changing the server code, update the deployment through **Deploy → Manage deployments → Edit → New version → Deploy**. Saving code in the editor alone does not update the deployed version.

See Google’s [web app documentation](https://developers.google.com/apps-script/guides/web) for deployment details.

## How Saving Works

Each participant receives a random ID. Submissions use that ID to update one row in `Responses`, rather than adding a row for every section.

The first two sections save in the background so participants can continue. Requests are sent sequentially; if a newer submission is waiting, it can replace an older pending submission because it includes the answers from all reached sections. The taskbar displays the save status. Final completion and early screening wait for confirmation before showing the thank-you window.

Failed submissions are retried automatically, up to three attempts in total. If the final submission still fails, a retry window appears and the answers remain available in the page.

The sheet records the participant ID, start and finish times, name, age, age group, gender, answers to questions 2–16, and completion status:

| Status | Meaning |
| --- | --- |
| `in_progress` | A partial response has been saved |
| `screened_out` | The participant finished through the screening branch |
| `complete` | The final response has been saved |

Answers are held in page memory, without using localStorage, sessionStorage, or cookies for response storage. Closing or refreshing the page loses unsent answers; responses already saved in Google Sheets remain there.

The confirmation endpoint returns save metadata, not names or answers. Keep the response sheet restricted to the research team and keep participant data out of the repository.

## Publish with GitHub Pages

1. Upload `index.html`, `style.css`, `app.js`, and this README to the repository root.
2. Open **Settings → Pages**.
3. Under **Build and deployment**, choose **Deploy from a branch**, then select `main` (or your chosen branch) and **/(root)**. Save.
4. Open the published URL and submit a test response. Check that the same row updates through completion.

`Code.gs` runs in Apps Script, not GitHub Pages. The frontend has no build command.

GitHub’s [publishing guide](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) covers the available settings.

## Checks Before Sharing

- Test empty welcome fields, age 11, and age 12.
- Check required answers, previous tabs, and locked future tabs.
- Verify the screening branch in question 3, the conditional question 5, and the exclusive option in question 6.
- Check that background saves update one row and that final completion sets `complete` and a finish time.
- Disconnect briefly, then reconnect and test the final retry flow.
- Check the separate answers icon, window close buttons, and downloaded Arabic text.
- Test mute, keyboard navigation, reduced motion, and a 360px mobile viewport.
- Check WhatsApp and link copying on the hosted site. Shared links should contain no participant data.

## Current Limitations

Saving depends on the network and Google Apps Script response time, so final confirmation can still take several seconds. The latest queue changes have passed local simulation checks; hosted testing is still needed.

Native sharing appears only in browsers that support it. The thank-you sign video panel has been removed from this version.
