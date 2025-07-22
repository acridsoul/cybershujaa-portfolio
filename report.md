# Cybersecurity Report: Studocu.com Document Access Vulnerability

## Problem Statement
Studocu.com restricts access to premium documents using overlays, CSS blurring, and download prevention. The "Studocuhack" browser extension exposes a vulnerability that allows users to bypass these restrictions and download premium content without authentication or payment.

## Approach & Tools Used
The extension exploits client-side weaknesses by:
- Removing premium banners and overlays
- Unblurring document pages
- Monitoring for dynamic restrictions
- Enabling document download via print

**Tools:** JavaScript DOM APIs, MutationObserver, CSS injection, Browser Extension APIs.

## Key Code Samples

**1. Remove Premium Banners:**
```js
const premiumBanner = document.querySelector('[class*="PremiumBannerBlobWrapper_preview-banner"]');
if (premiumBanner) premiumBanner.remove();
```
*Removes the paywall overlay, exposing the document.*

**2. Unblur Document Pages:**
```js
document.querySelectorAll('.page-content').forEach(page => {
    page.style.filter = 'none';
});
```
*Removes CSS blur, making premium content readable.*

**3. Real-Time DOM Monitoring:**
```js
const observer = new MutationObserver((mutations) => {
  mutations.forEach((mutation) => {
    mutation.addedNodes.forEach((node) => {
      if (node.nodeType === Node.ELEMENT_NODE && node.matches('[class*="PremiumBannerBlobWrapper"]')) {
        setTimeout(removePremiumBanner, 100);
      }
    });
  });
});
observer.observe(document.body, { childList: true, subtree: true });
```
*Removes dynamically loaded restrictions as soon as they appear.*

**4. Enable Document Download:**
```js
function createDownloadButton() {
    let downloadBtn = document.createElement("button");
    downloadBtn.addEventListener('click', generatePDF);
    return downloadBtn;
}
```
*Injects a button to generate a print-friendly, downloadable version of the document.*

## Lessons Learned
Client-side controls are insufficient for protecting premium content. Any content delivered to the browser can be accessed by users, regardless of UI restrictions. Robust server-side access control is essential. 