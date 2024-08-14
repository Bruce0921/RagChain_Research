# Week5 Report on Different Model Queries and Google Calendar API research

## **1. Introduction**

This report explores the integration of the Google Calendar API with a chatbot. This report will delve into the exact steps required to work with the Google Calendar API, with a particular focus on the process of integration, while maintaining a flexible approach towards the application format, whether it ends up as a web app or another form.

## **2. Prerequisites**

Before proceeding with the integration, certain prerequisites must be established to enable smooth interaction with Google Calendar:

- **Google Cloud Project**: A Google Cloud project must be created, and the Google Calendar API enabled.
- **OAuth 2.0 Credentials**: OAuth 2.0 credentials, including the Client ID and Client Secret, are set up within the Google Cloud Console.
- **Rag Chain App Setup**: The chatbot is developed using the Rag Chain framework, and FastAPI may be used to handle backend processes, though the exact format remains under exploration.

## **3. Setting Up Google Cloud Project and OAuth 2.0**

### 3.1 Creating a Google Cloud Project
The initial step involves setting up a project within the Google Cloud Console:

1. **Access Google Cloud Console**: The process begins by navigating to [Google Cloud Console](https://console.cloud.google.com/).
2. **Create a New Project**: A new project is created by selecting "New Project" from the project dropdown. A suitable name for the project is chosen before clicking "Create."
3. **Enable Google Calendar API**: The Google Calendar API is enabled by searching for it within the API & Services section and activating it.

### 3.2 Creating OAuth 2.0 Credentials
OAuth 2.0 credentials are essential for authenticating the application’s access to Google Calendar:

1. **Navigating to the Credentials Page**: The credentials are managed under the "Credentials" tab found within the API & Services section.
2. **Creating OAuth 2.0 Client ID**: 
   - The "Create Credentials" button is selected, followed by choosing "OAuth 2.0 Client ID."
   - The consent screen is configured, with specific attention to adding necessary scopes such as `https://www.googleapis.com/auth/calendar`.
   - The application type is set to "Web Application," though this choice may be revisited as the project’s format becomes more defined. Authorized redirect URIs are added, ensuring they align with those used in the Rag Chain app.
3. **Downloading and Understanding Credentials**: 
   - The credentials are downloaded in a `credentials.json` file. This file contains the Client ID and Client Secret, which are pivotal in the authentication process.
   - The Client ID uniquely identifies the application requesting access, while the Client Secret is used to authorize that request. These credentials must be stored securely, as they are used in the OAuth flow to exchange for tokens that allow interaction with the Google Calendar API.

### 3.3 Understanding OAuth 2.0 Flow
OAuth 2.0 serves as the mechanism through which users grant permission for the application to access their Google Calendar on their behalf. The following exploration outlines how this flow is managed:

1. **Requesting Authorization**: The application directs the user to Google's authorization server, where they log in and grant permission for access to their calendar data.
2. **Handling Redirect and Authorization Code**: Upon granting permission, Google redirects the user back to the application using the redirect URI provided earlier, along with an authorization code.
3. **Exchanging Authorization Code for Tokens**: This code is exchanged for access and refresh tokens, which the application will use to make authorized requests to the Google Calendar API. The access token is used to access the API, while the refresh token is used to obtain a new access token when the current one expires.

## **4. Working with OAuth 2.0 Authentication in the Application**

### 4.1 Configuring Environment Variables
The application needs to securely store and manage the OAuth credentials:

1. **Storing the Credentials**: The `credentials.json` file is placed in the root directory of the application or another secure location.
2. **Setting Environment Variables**: Environment variables are configured to hold the path to the credentials file, Client ID, and Client Secret, ensuring they are easily accessible but remain secure.

### 4.2 Implementing OAuth Flow
In order to authenticate and authorize access to the Google Calendar API, the OAuth flow is implemented as follows:

1. **Installing Necessary Libraries**:
   - The required Python libraries are installed using pip:
     ```bash
     pip install --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib
     ```
2. **Handling the OAuth Flow**:
   - The following code illustrates the initialization and handling of the OAuth flow:
     ```python
    import datetime
    import os.path

    from google.auth.transport.requests import Request
    from google.oauth2.credentials import Credentials
    from google_auth_oauthlib.flow import InstalledAppFlow
    from googleapiclient.discovery import build
    from googleapiclient.errors import HttpError

    # If modifying these scopes, delete the file token.json.
    SCOPES = ["https://www.googleapis.com/auth/calendar.readonly"]


    def main():
    """Shows basic usage of the Google Calendar API.
    Prints the start and name of the next 10 events on the user's calendar.
    """
    creds = None
    # The file token.json stores the user's access and refresh tokens, and is
    # created automatically when the authorization flow completes for the first
    # time.
    if os.path.exists("token.json"):
        creds = Credentials.from_authorized_user_file("token.json", SCOPES)
    # If there are no (valid) credentials available, let the user log in.
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
        creds.refresh(Request())
        else:
        flow = InstalledAppFlow.from_client_secrets_file(
            "credentials.json", SCOPES
        )
        creds = flow.run_local_server(port=0)
        # Save the credentials for the next run
        with open("token.json", "w") as token:
        token.write(creds.to_json())

    try:
        service = build("calendar", "v3", credentials=creds)

        # Call the Calendar API
        now = datetime.datetime.utcnow().isoformat() + "Z"  # 'Z' indicates UTC time
        print("Getting the upcoming 10 events")
        events_result = (
            service.events()
            .list(
                calendarId="primary",
                timeMin=now,
                maxResults=10,
                singleEvents=True,
                orderBy="startTime",
            )
            .execute()
        )
        events = events_result.get("items", [])

        if not events:
        print("No upcoming events found.")
        return

        # Prints the start and name of the next 10 events
        for event in events:
        start = event["start"].get("dateTime", event["start"].get("date"))
        print(start, event["summary"])

    except HttpError as error:
        print(f"An error occurred: {error}")


    if __name__ == "__main__":
    main()
     ```
   - Configure the sample
    Above is an example that you could put in your working directory, create a file named quickstart.py.
    Include the following code in quickstart.py:

   - Run the sample
    In your working directory, build and run the sample:

    ```bash
    python3 quickstart.py
    ```
    The first time you run the sample, it prompts you to authorize access:
    If you're not already signed in to your Google Account, sign in when prompted. If you're signed in to multiple accounts, select one account to use for authorization.
    Click Accept.
    Your Python application runs and calls the Google Calendar API.

    Authorization information is stored in the file system, so the next time you run the sample code, you aren't prompted for authorization.

## **5. Integrating Google Calendar API with the Rag Chain App**

### 5.1 API Integration: Event Management Functions
Once authenticated, the integration focuses on managing events through the Google Calendar API:

1. **Creating Events**:
   - The following function demonstrates how to create events on a user's Google Calendar:
     ```python
     from googleapiclient.discovery import build

     def create_event(creds, summary, location, description, start_time, end_time):
         service = build('calendar', 'v3', credentials=creds)
         event = {
             'summary': summary,
             'location': location,
             'description': description,
             'start': {
                 'dateTime': start_time,
                 'timeZone': 'America/Los_Angeles',
             },
             'end': {
                 'dateTime': end_time,
                 'timeZone': 'America/Los_Angeles',
             },
         }
         event = service.events().insert(calendarId='primary', body=event).execute()
         print(f"Event created: {event.get('htmlLink')}")
     ```
   - This function leverages the authenticated credentials to create an event, specifying details such as the event summary, location, description, and time. Upon successful creation, the function prints a link to the event.

2. **Fetching Events**:
   - Retrieving events from the calendar is accomplished with the following function:
     ```python
     def list_events(creds, max_results=10):
         service = build('calendar', 'v3', credentials=creds)
         events_result = service.events().list(calendarId='primary', maxResults=max_results, singleEvents=True, orderBy='startTime').execute()
         events = events_result.get('items', [])
         return events
     ```
   - This function queries the Google Calendar API to return a list of upcoming events, which can be displayed to the user or further processed by the application.

### 5.2 Handling User Interactions with the Chatbot
With the API integrated, the focus shifts to how the chatbot processes user requests and interacts with the Google Calendar:

1. **NLP Integration**:
   - Natural Language Processing (NLP) is utilized to interpret user inputs and map them to corresponding actions. For instance, a user input such as "Book an appointment with Dr. Smith next Monday at 3 PM" is parsed to extract details, which are then passed to the `create_event` function.

2. **Responding to Users**:
   - The chatbot provides feedback after actions are taken. For example, after scheduling an appointment, the chatbot might respond with:
     ```python
     response = f"Appointment with Dr. Smith has been scheduled for {start_time}."
     ```
   - This immediate feedback enhances the user experience by confirming that the desired action was successfully completed.

### 5.3 Addressing Edge Cases
Given the potential for various scenarios, it’s essential to consider and handle edge cases:

1. **Error Handling**:
   - Errors such as API call failures or user permission denial must be managed. Implementing robust error handling ensures that the chatbot can gracefully inform the user of any issues and possibly suggest corrective actions.

2. **Token Management**:
   - The OAuth tokens must be managed effectively, with mechanisms in place to refresh tokens when they expire, ensuring that the application maintains continuous access to the Google Calendar API.

## **6. Testing and Deployment**

### 6.1 Testing in a Development Environment
Thorough testing is required to validate the integration:

1. **Testing Calendar API Integration**:
   - Simulating various user interactions can help identify potential issues. Testing should cover scenarios like creating, updating, and fetching events to ensure that the API behaves as expected in all cases

### 6.2 Deploying the Application
Once testing is complete, the application is ready for deployment:

- Deploying the Chatbot:
    The chatbot is deployed to the chosen platform, whether it's a web application or another format. Security and stability are key considerations during deployment.
- Monitoring Post-Deployment:
    Continuous monitoring post-deployment is necessary to quickly address any issues that arise and to ensure that the application remains responsive and reliable.