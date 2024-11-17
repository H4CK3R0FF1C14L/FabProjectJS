# FabProjectJS

This project is a rewritten JavaScript copy of my [C# project](https://github.com/H4CK3R0FF1C14L/FabProject) which is created to add all free Quixel Megascans to the [FAB](https://fab.com) user library

# How to use
- Open [FAB](https://fab.com)
- Open Inspector -> Console
- Copy and Paste this code and wait when process done

```js
(async () => {
    const baseUrl = "https://www.fab.com";

    // Extract cookies and CSRF token from document cookies or other mechanisms
    const csrfToken = getCookie("fab_csrftoken");
    const cookie = document.cookie;

    if (!csrfToken || !cookie) {
        console.log("CSRF token or cookie is missing.");
        return;
    }

    console.log(`Token: ${csrfToken}`);
    console.log(`Cookie: ${cookie}\n\n`);

    await parseAllModels();

    async function parseAllModels() {
        let url = `${baseUrl}/i/listings/search?currency=USD&seller=Quixel`;

        do {
            const response = await fetch(url, {
                method: "GET",
                headers: getHeaders()
            });

            if (!response.ok) {
                console.error("Failed to fetch data.");
                return false;
            }

            const data = await response.json();
            if (!data || !data.results || data.results.length === 0) return false;

            for (const result of data.results) {
                if (!result.uid) continue;

                const offerId = await getOfferId(result.uid);
                if (!offerId) continue;

                await addToLibrary(result.uid, offerId);
            }

            url = data.next || null;
        } while (url);

        return true;
    }

    async function getOfferId(uid) {
        const response = await fetch(`${baseUrl}/i/listings/${uid}`, {
            method: "GET",
            headers: getHeaders()
        });

        if (!response.ok) return null;

        const itemData = await response.json();
        return itemData?.licenses?.[0]?.offerId || null;
    }

    async function addToLibrary(uid, offerId) {
        const url = `${baseUrl}/i/listings/${uid}/add-to-library`;

        const headers = getHeaders({
            "X-CsrfToken": csrfToken,
            "Origin": baseUrl,
            "Referer": `${baseUrl}/listings/${uid}`
        });

        const formData = new FormData();
        formData.append("offer_id", offerId);

        const response = await fetch(url, {
            method: "POST",
            headers: headers,
            body: formData
        });

        if (response.ok) {
            console.log(`Added item ${uid} to library.`);
        } else {
            console.error(`ERROR: ${response.status}`);
        }
    }

    function getHeaders(customHeaders = {}) {
        return {
            "Accept": "application/json, text/plain, */*",
            "Connection": "keep-alive",
            "Cache-Control": "max-age=0",
            "DNT": "1",
            "User-Agent": navigator.userAgent,
            "Cookie": cookie,
            ...customHeaders
        };
    }

    function getCookie(name) {
        const match = document.cookie.match(new RegExp(`(^| )${name}=([^;]+)`));
        return match ? match[2] : null;
    }
})();
```

# Note
I am not a JavaScript programmer so I can't vouch for the quality and workability of this shit code, if you have any problems, I don't plan to solve them.
