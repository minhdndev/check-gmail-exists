# Gmail Exists Validator

This Python script checks whether a given Gmail account exists by attempting to access its public Google Calendar ICS feed.

## How It Works
- The script constructs a Google Calendar ICS URL using the given Gmail address.
- It sends an asynchronous HTTP request to the URL.
- If the response contains the header `x-frame-options` with the value `sameorigin`, the script assumes that the Gmail account exists.

# Example code
```py
import aiohttp
import asyncio
from aiolimiter import AsyncLimiter
from typing import Optional

class GmailExistsValidator:
    def __init__(self, rate_limit: int = 1000):
        """
        :param rate_limit: Minimum time (milliseconds) between requests
        """
        self.rate_limiter = AsyncLimiter(1, rate_limit / 1000)  # Convert to seconds

    async def email_exists(self, email: str) -> Optional[bool]:
        """Check if Gmail account exists"""
        url = f"https://calendar.google.com/calendar/ical/{email}/public/basic.ics"
        
        async with self.rate_limiter:  # Rate limit requests
            async with aiohttp.ClientSession() as session:
                try:
                    async with session.get(url) as response:
                        headers = response.headers
                        
                        if 'x-frame-options' in headers:
                            if headers['x-frame-options'].lower() == 'sameorigin':
                                print(f"Gmail account ({email}) exists")
                                return True
                except aiohttp.ClientError as e:
                    print(f"Request error for {email}: {e}")
                    return None
        
        print(f"Gmail account ({email}) does NOT exist")
        return False

# Example usage
async def main():
    validator = GmailExistsValidator()
    email_list = [
        "example@gmail.com",
        "example2@gmail.com",
        "example3@gmail.com",
        "example4@gmail.com",
        "example5@gmail.com",
        "example6@gmail.com",
        "example7@gmail.com",
    ]
    for email in email_list:
        is_gmail_exists = await validator.email_exists(email)
        print(f"Email {email} exists: {is_gmail_exists}\n")

if __name__ == "__main__":
    asyncio.run(main())
```

## Notes
- This method relies on an indirect technique to check Gmail account existence and may not be 100% accurate.
- Google may change its services, affecting the validity of this approach.

## Purposes
This project is for educational purposes only.

