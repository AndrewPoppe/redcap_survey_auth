# REDCap Survey Auth

A REDCap External Module that adds authentication to surveys.

## Purpose / Use Case

In some cases it may be useful to present users with a data entry form but not confront them with the REDCap user interface, yet still be able to tell who the person entering the data was. This way, they do not need to be members of the project or even have a REDCap account. 

Possible use cases may be incident reports, internal orders or requests for goods or services, etc.

## Effect

When enabled for a survey, a login page will be displayed as first page of a survey that the user needs to complete before being able to proceed to the survey (similar to the reCAPTCHA feature introduced in REDCap 8.11). 

Survey users can be authenticated against REDCap Users (table-based authentication), any number or LDAP servers, and/or against a list or username/password entries provided in the module's project configuration.

## Limitations

- Currently, there is no mechanism that limits login attempts. 

## Requirements

REDCAP 8.1.0 or newer (tested with REDCap 8.11.9 on a system running PHP 7.0.33).

## Installation

- Clone this repo into <redcap-root>/modules/redcap_survey_auth_v<version-number>, or
- Obtain this module from the Consortium REDCap Repo via the Control Center.
- Go to Control Center > Technical / Developer Tools > External Modules and enable REDCap Survey Auth.
- Enable the module for each project that needs survey authentication. Be sure to include the action tag @SURVEY-AUTH somewhere in the survey instrument and to configure the module's project settings (authentication is deactivated with the default settings).

## Configuration

### System-Level Settings

- **Global - Debug Mode:** When enabled, some additional information will be printed to the Web-browser's console. This setting is global, i.e. debug info will be shown regardless of the debug setting in a project.

- **Lockout time:** The time, in (whole) minutes, a user (based on client IP) is denied further login attempts. Defaults to 5 minutes. Explicitly setting this to 0 (zero) will disable the lockout mechanism.

### Project-Level Settings

- **Logging:** Determines the type of logging that occurs.
  - _None:_ No log entries will be produced.
  - _Failed attempts only:_ Log entries will be produced for failed login attempts only.
  - _Successful attempts:_ Log entries will be produced for successful logins.
  - _All:_ Log entries will be produced for both types of events.

- **Debug Mode:** When enabled, some additional information that may help troubleshooting will be printed to the Web-browser's console.

- **API Token:** An API token of a user who can create new records. This is required, because the module needs to create a new record (and set data for any of the mappings defined in the action tag - see below) in order to be able to forward the survey user to the actual survey after authentication.

- **Text displayed above username/password fields:** Optionally enter some prompt that is displayed to the survey user.

- **Username label:** The label to be displayed for the username text box. Defaults to 'Username'.

- **Password label:** The label to be displayed for the password box. Defaults to 'Password'.

- **Submit label:** The label to be displayed on the submit button. Defaults to 'Submit'.

- **Fail message:** A message that is displayed to the user in case the login fails. Defaults to 'Invalid username and/or password or access denied.'.

- **Lockout message:** A message that is displayed to the user in case of too many failed login attemptss. Defaults to 'Too many failed login attempts. Please try again later.'.

- **Authentication methods:** Any of the following methods can be used for authentication. Note the order, in which authentication is attempted: Custom &gt; Table &gt; Other LDAP &gt; LDAP. 

  - **Table:** The REDCap user table is used to look up user/password combinations.

  - **LDAP:** When REDCap is set to use LDAP, this is used for authentication.

  - **Other LDAP:** Provide any number of LDAP connection info as a JSON array. The order of processing will be as provided in the array. See the [PEAR manual for the LDAP auth container](https://pear.php.net/manual/en/package.authentication.auth.storage.ldap.php) for a list of parameters (_debug_ is not supported) or the LDAP configuration in REDCap's `webtoolsl2/ldap` folder.

    Example:

    ```JSON
    [
        {
            "url": "ldap://127.0.0.1",
            "port": 389,
            "version": 3,
            "binddn": "bindUsername",
            "bindpw": "bindpassword",
            "attributes": [],
            "userattr": "samAccountName",
            "userfilter": "(objectCategory=person)",
            "start_tls": false,
            "referrals": false
        }
    ]
    ```

  - **Custom:** When selected, custom credentials can be entered into a text box. Type one username-password pair per line, separated by a colon (e.g. `UserXY:secret123`). Usernames are not case-sensitive (passwords are).

- **Use Whitelist:** When checked, a list of usernames (one username per line) can be entered. Only users in this list will be able to authenticate successfully. 

### @SURVEY-AUTH Action Tag

To enable authentication for a survey, the **@SURVEY-AUTH** action tag must be used on any field of the survey instrument. There must be at most one action tag per instrument.

```ActionTag
@SURVEY-AUTH(success=value, username=fieldname, email=fieldname, fullname=fieldname, timestamp=fieldname)
```

When _username_, _email_, _fullname_, or _timestamp_ are defined, the respective data will be inserted into to records into the specified fields. Timestamp format will match the datetime format of the target field (time-only fields are not supported; defaults to YMD).

When a value for _success_ is defined, the field with the action tag will be set to this value. If used, the field should be set to @READONLY/@READONLY-SURVEY or @HIDDEN-SURVEY.
