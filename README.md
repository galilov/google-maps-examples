# Google Maps for Capacitor Examples

## How to Run

- Run `npm install` in the root directory.
- Change into the UI Framework directory of choice. (e.g. `cd vue3`)
- Run `npm install`.
- Build the web app: `npm run build`.
- Change back into the root folder (`cd ..`)
- Copy the built web app into the native platform: `npx cap copy`.
- Update the native plugins and dependencies referenced in package.json: `npx cap update`.*
- Make sure you've added your API-keys. Replace the occurrences of `your_api_key` with your API-key.
  - Android: In `AndroidManifest.xml` look for `your_api_key`.
  - iOS: in the `src` folder (within the UI Framework directory of choice) search for `your_api_key`.
  - For info about obtaining API-keys, check out the [documentation](https://capacitor-community.github.io/google-maps/#/getting-started/installation?id=obtain-api-keys)
- Open the native IDE and run the app: `npx cap open android` or `npx cap open ios`.

* This step is only needed after running `npm install`. So probably only the first time you clone this repository.
