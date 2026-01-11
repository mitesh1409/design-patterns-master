# Design Patterns -> Structural -> Proxy

Proxy is a structural design pattern that lets you provide a substitute or placeholder for another object. A proxy controls access to the original object, allowing you to perform something either before or after the request gets through to the original object.

The Proxy design pattern is like having a **personal assistant**. Instead of talking directly to a busy manager (the "Real Subject"), you talk to the assistant (the "Proxy"). The assistant can handle simple requests, screen calls, or wait until the manager is actually free before passing a message along.

In programming, a Proxy is a structural design pattern that lets you provide a substitute or placeholder for another object. A proxy controls access to the original object, allowing you to perform something either before or after the request reaches the original object.

## Problem

Imagine you have a massive, resource-intensive object—like a database service or a high-definition video downloader.

If you initialize this object as soon as the application starts, it consumes a lot of memory and processing power, even if the user never actually clicks the "Download" button. Furthermore, if multiple parts of your code need to use this service, you might end up recreating it several times or leaving it unprotected.

**Problematic Code Example**  

In this example, the `VideoService` is initialized immediately, even if we don't know if the user will ever use it.

```typescript
// The heavy service
class VideoService {
    constructor() {
        console.log("Loading heavy video files from the server... (Slow)");
    }

    renderVideo(id: string) {
        console.log(`Displaying video content for ID: ${id}`);
    }
}

// Client code
const service = new VideoService(); // The "heavy" work happens here immediately
// ... maybe the user never actually calls renderVideo()
```

---

## Solution

The Proxy pattern suggests creating a new class with the **same interface** as the original service. You then update your app so that it communicates with the Proxy instead of the real object.

When the Proxy receives a request, it can:

1. **Lazy Initialization:** Wait until the very last second to create the "heavy" object.
2. **Access Control:** Check if the user has permission.
3. **Caching:** Check if the result is already saved so it doesn't have to call the heavy service again.

To implement this, both the Real Subject and the Proxy should follow the same interface. This way, the client doesn't even know they are talking to a proxy.

**Example #1**  

```typescript
// 1. The common interface
interface IVideoService {
    renderVideo(id: string): void;
}

// 2. The Real Subject (The heavy object)
class RealVideoService implements IVideoService {
    constructor() {
        this.heavyInitialLoad();
    }

    private heavyInitialLoad() {
        console.log("🔥 System: Loading 5GB of video metadata...");
    }

    renderVideo(id: string) {
        console.log(`🎬 System: Rendering video ${id}`);
    }
}

// 3. The Proxy
class VideoProxy implements IVideoService {
    private realService: RealVideoService | null = null;

    renderVideo(id: string) {
        // Lazy Initialization: Only create the real service when actually needed
        if (this.realService === null) {
            this.realService = new RealVideoService();
        }

        console.log(`🛡️ Proxy: Logging access request for ${id}...`);
        this.realService.renderVideo(id);
    }
}

// CLIENT CODE

console.log("--- App Starting ---");
const proxy = new VideoProxy(); 
// Notice: No "Heavy Loading" message yet!

console.log("--- User browsing the menu ---");
// ... time passes ...

console.log("--- User clicks 'Play Video' ---");
proxy.renderVideo("101"); 
// ONLY NOW: The heavy service is created and the video is rendered.
```

### Key Benefits

* **Performance:** You save memory by not loading heavy objects until they are needed.
* **Security:** You can add a check inside the proxy to see if a user is "Admin" before calling the real service.
* **Clean Code:** The original service stays focused on its main job, while the proxy handles the "meta" tasks like logging or caching.

---

**Example #2**

Debit Card, Credit Card, UPI etc. are Proxy for the real Bank Account.
