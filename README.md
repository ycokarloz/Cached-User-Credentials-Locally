# Cached-User-Credentials-Locally

# Overview 
Windows systems, by default, cache a certain number of user credentials (passwords) locally. This feature is primarily designed to allow users to log into their machines when the machine cannot communicate with the domain controller, which is common when working remotely without internet access or, as in your case, when the device is in airplane mode.

When the domain controller locks an account due to multiple failed login attempts (a common security measure to prevent brute force attacks), this status is not immediately communicated to or enforced by the local cache on your machine. If the device is offline (e.g., in airplane mode), it relies solely on the cached credentials for authentication. As a result, even if the account is locked on the server, a user can still log in using the cached credentials on their local machine if they enter the correct password.
# Proof of Concept
## Info:
Domain Server IP: 192.168.0.1
Client Computer IP: 192.168.0.10 
## Client Side
Currently logged in with a domain user, `pbpeter`.
The client can ping the domain server at 192.168.0.1 without interruptions.
![Pasted image 20240429154427](https://github.com/ycokarloz/Cached-User-Credentials-Locally/assets/92008086/da8e40fb-8534-4c5a-a798-663a3069536f)
Now we remove the client from connecting to its Domain Server for testing. (Also this is similar with using Airplane mode).
![Pasted image 20240429154543](https://github.com/ycokarloz/Cached-User-Credentials-Locally/assets/92008086/ca44578a-a800-4604-9de2-529d472caf76)

As you can see, we are not able to ping the Domain Server.
We will verify if it's possible to log in to the account despite the client's device being disconnected from its domain server.
Testing the login process while the connection to the Domain Server is unavailable.
![Pasted image 20240429154649](https://github.com/ycokarloz/Cached-User-Credentials-Locally/assets/92008086/687a227d-38c2-4a5b-81f2-a628541c0cd2)

I managed to log in even though I'm no longer connected to the Domain Server. This is due to the cached credentials stored on the device.
![Pasted image 20240429154807](https://github.com/ycokarloz/Cached-User-Credentials-Locally/assets/92008086/9e7c5f3f-513e-4d20-a2fe-6aa5d1013ce5)

Now, let's implement measures to mitigate the risk.
## Mitigation - Server Side
Navigate to Group Policy Management. 
Choose the GPO for editing.
![Pasted image 20240429153657](https://github.com/ycokarloz/Cached-User-Credentials-Locally/assets/92008086/aab771b4-0309-4438-b70d-de37af497079)

Navigate to `Computer Configuration` -> `Windows Settings` -> `Security Settings` -> `Local Policies` -> `Security Options > Interactive logon: Number of previous logons to cache (in case domain controller is not available)`
![Pasted image 20240429153850](https://github.com/ycokarloz/Cached-User-Credentials-Locally/assets/92008086/2416a2a0-4a18-4607-a3a2-b1ad75f7e7a6)

Check the `Define this policy setting`.
Set the value to 0.
![Pasted image 20240429153958](https://github.com/ycokarloz/Cached-User-Credentials-Locally/assets/92008086/ab5d7bfb-038e-4db1-891a-61d307438887)

Value 0 disables caching.
## Mitigation - Client Side
Go to `Local Security Policy > Local Policies > Security Options > Interactive logon: Number of previous logons to cache (in case domain controller is not available)`
![Pasted image 20240429151805](https://github.com/ycokarloz/Cached-User-Credentials-Locally/assets/92008086/8de85c2b-bf00-41f8-b257-3def0b2b0dff)

Set the value to 0. 
Value 0 disables caching.
## Let's Test again
Currently, I am still disconnected on the Domain Server.
I'll attempt to log in once more using the same account while mitigations are implemented and disconnected from the Domain Server.
![Pasted image 20240429155044](https://github.com/ycokarloz/Cached-User-Credentials-Locally/assets/92008086/d37e4744-0183-43d1-baf5-901d924336d9)

Now, the user is unable to log in even while disconnected from their local domain server.

# Conclusion
In conclusion, the issue with cached credentials in a domain environment presents both a challenge and an operational necessity. While credential caching allows users to continue working even when disconnected from the domain server, it can also pose a significant security risk, particularly if an unauthorized user gains access to a device with valid cached credentials.

However, completely disabling cached credentials might impact user productivity in scenarios where domain server connectivity is temporarily unavailable. Therefore, a balanced approach is necessary. Organizations should consider their specific needs and potential security threats when configuring cached logon settings. Employing additional security measures such as multi-factor authentication can also help mitigate risks without severely impacting user access and productivity.
