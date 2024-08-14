# Week5 Report on Different Model Queries and Google Calendar API research

## **1. Introduction**

This report explores the integration of Google Calendar API with a chatbot to streamline the management of medical appointments. The integration will allow users to book appointments, set reminders, and manage follow-up visits through natural language interactions with the chatbot.

## **2. Prerequisites**

Before proceeding with the integration, ensure that the following prerequisites are met:

- **Google Cloud Project**: A Google Cloud project is created and the Google Calendar API is enabled.
- **OAuth 2.0 Credentials**: OAuth 2.0 credentials (Client ID and Client Secret) are set up in the Google Cloud Console.
- **Rag Chain App Setup**: The chatbot is built using the Rag Chain framework, and FastAPI is used to handle backend processes.

## **3. Setting Up Google Cloud Project and OAuth 2.0**

### 3.1 Create a Google Cloud Project
1. **Go to Google Cloud Console**: Navigate to [Google Cloud Console](https://console.cloud.google.com/).
2. **Create a New Project**: Click on the project dropdown and select "New Project." Name your project and click "Create."
3. **Enable Google Calendar API**: In the API & Services section, search for "Google Calendar API" and enable it.

### 3.2 Create OAuth 2.0 Credentials
1. **Navigate to the Credentials Page**: Go to the "Credentials" tab under the API & Services section.
2. **Create OAuth 2.0 Client ID**: 
   - Select "Create Credentials" and choose "OAuth 2.0 Client ID."
   - Configure the consent screen, including adding scopes like `https://www.googleapis.com/auth/calendar`.
   - Set the application type to "Web Application" and add authorized redirect URIs. (Ensure these match the redirect URIs in your Rag Chain app.)
3. **Download Credentials**: Download the `credentials.json` file containing your Client ID and Secret. Store this file securely.

## **4. Implementing OAuth 2.0 Authentication in Your App**

### 4.1 Setting Up Environment Variables
1. **Store Credentials**: Place the `credentials.json` in your application's root directory or another secure location.
2. **Environment Configuration**: Set up environment variables to hold the path to the credentials file, client ID, and client secret.

### 4.2 Implementing OAuth Flow
1. **Install Necessary Libraries**:
   - Install the required Python libraries using pip:
     ```bash
     pip install google-auth google-auth-oauthlib google-auth-httplib2 google-api-python-client
     ```
2. **Initialize OAuth Flow**:
   - Use the following code to handle the OAuth flow:
     ```python
     from google_auth_oauthlib.flow import InstalledAppFlow
     from google.auth.transport.requests import Request
     import os
     import pickle

     SCOPES = ['https://www.googleapis.com/auth/calendar']

     def authenticate_google():
         creds = None
         if os.path.exists('token.pickle'):
             with open('token.pickle', 'rb') as token:
                 creds = pickle.load(token)

         if not creds or not creds.valid:
             if creds and creds.expired and creds.refresh_token:
                 creds.refresh(Request())
             else:
                 flow = InstalledAppFlow.from_client_secrets_file('path/to/credentials.json', SCOPES)
                 creds = flow.run_local_server(port=0)

             with open('token.pickle', 'wb') as token:
                 pickle.dump(creds, token)

         return creds
     ```

## **5. Integrating Google Calendar API with Rag Chain App**

### 5.1 Event Management Functions
1. **Creating Events**:
   - Implement a function to create events on the user's Google Calendar:
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

2. **Fetching Events**:
   - Implement a function to fetch upcoming events:
     ```python
     def list_events(creds, max_results=10):
         service = build('calendar', 'v3', credentials=creds)
         events_result = service.events().list(calendarId='primary', maxResults=max_results, singleEvents=True, orderBy='startTime').execute()
         events = events_result.get('items', [])
         return events
     ```

### 5.2 Integrating with Chatbot
1. **NLP Integration**:
   - Use NLP techniques to parse user inputs and determine the action to take (e.g., booking an appointment).
   - For example, if the user says, "Book an appointment with Dr. Smith next Monday at 3 PM," the chatbot should extract relevant details and call `create_event`.

2. **Chatbot Response**:
   - After performing an action, provide feedback to the user:
     ```python
     response = f"Appointment with Dr. Smith has been scheduled for {start_time}."
     ```

### 5.3 Handling Edge Cases
1. **Error Handling**:
   - Implement error handling to manage cases where the API call fails, or the user doesn’t grant permission.

2. **Token Refreshing**:
   - Ensure that the OAuth tokens are refreshed as needed to maintain continuous access.

## **6. Testing and Deployment**

### 6.1 Testing in Development Environment
1. **Test Calendar API Integration**:
   - Simulate different user interactions to ensure that the calendar functions are working as expected.
   - Validate that events are being created, updated, and fetched correctly.

### 6.2 Deploying the Application
1. **Deploy Chatbot**:
   - Deploy the chatbot to your preferred platform, ensuring it is secure and stable.
2. **Monitor Performance**:
   - Monitor the application post-deployment to catch any issues early and make necessary adjustments.

## **7. Conclusion**

By following the steps outlined in this report, your Rag Chain app will be able to efficiently manage medical appointments using the Google Calendar API. This integration enhances the functionality of the chatbot, making it a valuable tool for both healthcare providers and patients.
