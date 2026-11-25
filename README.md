# OrangeHRM Cypress E2E

Cypress end-to-end coverage for the OrangeHRM demo, implemented with Gherkin BDD scenarios and page objects.

## Requirements

- Node.js 18 or newer (Node.js 20 LTS recommended)
- npm

## Run the tests

Install dependencies:

```sh
npm install
```

Open Cypress in GUI mode and select `admin-users.feature` to run the suite:

```sh
npx cypress open
```

Run headlessly:

```sh
npx cypress run
```

The equivalent npm scripts are `npm run test:e2e:open` and `npm run test:e2e`.

The scenarios use the demo account (`Admin` / `admin123`) stored in `cypress/fixtures/credentials.json`; do not use a real account there. The admin-creation scenarios generate unique usernames from the `cypressuser1` and `cypressuser2` prefixes to avoid conflicts with prior runs. One scenario uses an invalid fixture password to verify the login error state.

## Latest verification

- Headless: 5 scenarios passed with Cypress 14.5.4 and Electron 130.
- GUI: Cypress opened successfully; select the feature in the app to execute it interactively.

## Results and artifacts

- Mochawesome HTML report: `test-artifacts/reports/index.html`
- Test evidence and failure screenshots: `test-artifacts/screenshots/admin-users.feature/`

The suite captures evidence for the dashboard, invalid-login validation, Admin user management, and each created user. The HTML report and screenshots are included in the repository; rerunning the headless suite refreshes them. Test results are environment-dependent because the suite targets the live OrangeHRM demo site.

## Jenkins local jobs (Week 11)

Jenkins must run on a machine that can access the OrangeHRM demo and has Node.js and npm available on the Jenkins agent's `PATH`. Cypress 14 also needs its supported browser/runtime dependencies. On Windows, install Jenkins using the official Windows installer and verify the Jenkins service account can run `node --version` and `npm --version`. For Linux agents, install the Cypress system dependencies and use a supported browser environment.

Install the Jenkins **Pipeline**, **Git**, and **HTML Publisher** plugins. The HTML Publisher plugin adds the Pipeline `publishHTML` step. Configure Jenkins' global security and an administrator account before exposing the service; do not expose an unauthenticated Jenkins instance to the internet.

### Freestyle Project

1. In Jenkins, select **New Item**, enter a job name such as `OrangeHRM-Cypress-Freestyle`, choose **Freestyle project**, and save.
2. Under **Source Code Management**, select **Git** and set the repository URL to `https://github.com/Geryongko/CypressOrange-Week11-Jenkins.git`. Set **Branch Specifier** to `*/main`. Jenkins checks out the repository into the job workspace; do not set the working directory to the repository's `.git` folder.
3. Ensure the checkout contains this assignment's `Jenkinsfile` if configuring the Pipeline job below. Under **Build Steps**, add **Execute Windows batch command** on Windows (or **Execute shell** on Linux/macOS). Clean stale outputs, install dependencies, and run tests. For Windows, use:

   ```bat
   @echo on
   if exist test-artifacts rmdir /s /q test-artifacts
   call npm ci
   if errorlevel 1 exit /b %errorlevel%
   call npm run test:e2e
   exit /b %errorlevel%
   ```

   For Linux/macOS, use:

   ```sh
   rm -rf test-artifacts/reports test-artifacts/screenshots
   npm ci
   npm run test:e2e
   ```

4. Under **Post-build Actions**, add **Archive the artifacts** and enter `test-artifacts/reports/**,test-artifacts/screenshots/**`. Enable **allow empty archive** if available so a setup failure does not hide the console log.
5. Save, select **Build Now**, open the build's **Console Output**, and capture a screenshot showing the Cypress summary. The archived report and screenshots are linked from the build page.

### Pipeline

1. Select **New Item**, enter a job name such as `OrangeHRM-Cypress-Pipeline`, choose **Pipeline**, and save.
2. Make sure the `Jenkinsfile` from this assignment is committed to the branch Jenkins will build. Under **Pipeline**, select **Pipeline script from SCM**, choose **Git**, set the repository URL to `https://github.com/Geryongko/CypressOrange-Week11-Jenkins.git`, set **Branch Specifier** to `*/main`, and set **Script Path** to `Jenkinsfile`.
3. Save and select **Build Now**. The Jenkinsfile checks out the repository, runs `npm ci` and `npm run test:e2e`, then always archives the HTML report and screenshots. If the **HTML Publisher** plugin is installed, a **Cypress HTML Report** link appears on the build page even when test execution fails but the report was generated.
4. Open **Console Output** and capture a screenshot showing the Cypress test summary. Open **Cypress HTML Report** and capture a screenshot of the report summary. The report and screenshot files are also downloadable from **Archived Artifacts**.

### Week 11 run evidence

The local Jenkins Freestyle job completed build **#4** successfully, and the Pipeline job completed build **#2** successfully. The Pipeline's Mochawesome report records **5 passing tests and 0 failures**. The report, its assets, and authentic build screenshots are included under `test-artifacts/jenkins-evidence/`:

- `reports/index.html` and the complete report bundle `mochawesome-report-pipeline-2.zip`
- `freestyle-console-build-4.png`
- `pipeline-console-build-2.png`
- `pipeline-mochawesome-report.png`

The Pipeline's **Cypress HTML Report** link is published by Jenkins. If the Jenkins report page appears blank, its default content-security policy is blocking the report's JavaScript; this does not mean the report or assets are missing. Extract the ZIP and open `index.html` locally to view the same report. Do not weaken Jenkins' global content-security policy solely to display this report. Cypress retries are enabled once in headless mode, so a transient failed attempt can still produce a passing final scenario result and an additional attempt screenshot.

## Coverage

- Fixture-backed login and dashboard redirect validation
- Admin sidebar navigation and user-management page assertions
- User creation for ESS and Admin roles, followed by a user-list search
- Negative login validation for invalid credentials
