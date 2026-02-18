# Postbook

## Room Info
This room is centered on practical web vulnerabilities. The flow rewards careful exploration of features, URLs, and how the app handles access controls.

## Initial Recon
Before looking at any hints, I tried to explore all features naturally. I inspected the homepage, checked the page source, and looked at the URL for parameters. Nothing interesting showed up on the landing page.

![ss-01](images/Postbook/ss-01.png)

## Account Creation and XSS Attempts
I created a new account. I attempted XSS during registration, but inputs only allow lowercase letters, so nothing useful there. After signing in, I inspected URLs and page source again—still nothing notable.

I created a random post for testing and tried basic XSS:

```html
<script>alert('XSS')</script>
```

No execution.

![ss-02](images/Postbook/ss-02.png)

## Edit Post IDOR (Flag 1)
I edited the post and noticed a parameter in the URL. That was a big signal. I changed the parameter to target another post and confirmed I could edit it.

![ss-03](images/Postbook/ss-03.png)

That exposed the first flag.

![ss-04](images/Postbook/ss-04.png)

This was **broken access control / IDOR** through the edit feature.

## Delete Post IDOR via Weak Identifier (Flag 2)
Since edit was vulnerable, I checked delete. The delete action immediately removes a post, so I inspected the delete URL by hovering/copying the link.

![ss-05](images/Postbook/ss-05.png)

The ID looked like a hash: `e4da3b7fbbce2345d7772b0674a318d5`. I checked on CrackStation and confirmed it was the MD5 of `5`.

![ss-06](images/Postbook/ss-06.png)

If delete uses MD5(id), then I should be able to delete other users’ posts by swapping in the MD5 of their IDs. I generated the MD5 for `1` using CyberChef: `c4ca4238a0b923820dcc509a6f75849b`.

![ss-07](images/Postbook/ss-07.png)

I edited the delete URL with that hash and got another flag.

![ss-08](images/Postbook/ss-08.png)

## Private Post Exposure via Edit (Flag 3)
I created a post meant to be private.

![ss-09](images/Postbook/ss-09.png)

With another account, I confirmed the post wasn’t visible on the homepage.

![ss-10](images/Postbook/ss-10.png)

But the edit vulnerability still applied. By plugging the private post’s ID into the edit URL, I could view and edit it, which revealed the next flag.

![ss-11](images/Postbook/ss-11.png)

At this point, I had 3 flags.

## Profile ID Parameter Check
I visited **My Profile** and saw another ID parameter in the URL.

![ss-12](images/Postbook/ss-12.png)

I tried changing it, which confirmed a weakness, but it didn’t reveal a new flag beyond what I’d already gained from the edit vulnerability.

## Hidden Field Tampering (Flag 4)
Next I went to **Write a new post**. I tested XSS again, but it didn’t work. While inspecting the page, I noticed a `Hidden` keyword.

![ss-13](images/Postbook/ss-13.png)

I changed it to `show`, which revealed another input field.

![ss-14](images/Postbook/ss-14.png)

This looked like an owner/user ID. I changed it, saved the post, and got another flag.

![ss-15](images/Postbook/ss-15.png)

## Hint: “user” Has an Easy Password (Flag 5)
The hint said the account `user` has a very easy password. Instead of brute force, I used what I already knew.

I went to my own **Settings** page and checked cookies.

![ss-16](images/Postbook/ss-16.png)

The ID was hashed again. I replaced it with the MD5 of `2` (the ID for user `user`) and reloaded.

![ss-17](images/Postbook/ss-17.png)

The username switched to `user`. The password was hidden, but inspecting the password field and changing `type="password"` to `text` revealed it.

![ss-18](images/Postbook/ss-18.png)

The password was literally `password`.

![ss-19](images/Postbook/ss-19.png)

I logged in and received another flag.

![ss-20](images/Postbook/ss-20.png)

## Hint: “189 * 5” (Flag 6)
The hint points to `945`. That looked like a post ID or user ID. Using the previously found IDOR techniques, I accessed it and got the flag.

![ss-21](images/Postbook/ss-21.png)

## Hint: Cookie-based Session (Flag 7)
The final hint suggested the cookie keeps you logged in. Since the cookie uses MD5 of the user ID, I replaced it with the MD5 of `1` and reloaded. That granted the final flag.

![ss-22](images/Postbook/ss-22.png)

As an extra, I checked the admin’s password via the same inspect trick: it was `winks`.

![ss-23](images/Postbook/ss-23.png)

## Summary
- **IDOR / Broken access control** via edit and delete functions.
- **Weak identifier** (MD5 of numeric IDs) used for delete and session.
- **Hidden field tampering** to post as another user.
- **Credential disclosure** through DOM inspection and cookie manipulation.
