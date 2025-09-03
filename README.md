# Frame.io Filename Copier

A tiny bookmarklet that extracts all filenames with common extensions from a [Frame.io](https://frame.io) review page and copies them to your clipboard as a newline-separated list.

## Installation
1. Copy the JavaScript snippet from [`bookmarklet.js`](./bookmarklet.js) or below.
2. Create a new bookmark in your browser.
3. Paste the snippet into the bookmark’s **URL/location** field.
4. Name it something like `Copy Frame.io Filenames`.

## Usage
1. Open a Frame.io review link in your browser.
2. Click the `Copy Frame.io Filenames` bookmarklet.
3. The script scans the page for filenames (`.mov`, `.mp4`, `.jpg`, etc.).
4. If filenames are found, they are copied to your clipboard (one per line).  
   - If no filenames are detected, you’ll see an alert.
  
## Notes

Only filenames with common extensions are collected.
Works on whatever is visible in the DOM at the time. If a folder is not opened in the review page, its contents will not be scraped.
Read-only: this script does not modify your Frame.io projects

## Code
```javascript
javascript:(()=>{const ex=/\.(mp4|mov|m4v|wav|mp3|aif|aiff|mxf|flac|avi|mkv|webm|pdf|jpg|jpeg|png|tif|tiff|gif|webp|heic|psd|ai|indd|zip|rar|7z|tar|gz|bz2|docx?|xlsx?|pptx?|csv)$/i;const host=location.host.replace(/^www\./,'');const S=new Set();const clean=s=>s.replace(/\s+/g,' ').replace(/^[^\w\[\]().-]+|[^\w\[\]().-\s]+$/g,'').trim();const looksLikeFile=s=>!!s&&ex.test(s.trim());const add=v=>{if(!v)return;const t=clean(v);if(!t)return;if(t.toLowerCase()===host.toLowerCase())return;if(looksLikeFile(t))S.add(t);};const fromHref=h=>{try{const url=new URL(h,location.href);const last=(url.pathname.split('/').pop()||'').split('?')[0];return looksLikeFile(last)?decodeURIComponent(last):'';}catch{return''}};const visit=n=>{if(!n)return;if(n.nodeType===Node.TEXT_NODE){add(n.textContent);}else if(n.nodeType===Node.ELEMENT_NODE){const el=n;['data-filename','data-name','title','aria-label','download'].forEach(a=>{const v=el.getAttribute&&el.getAttribute(a);if(v)add(v);});if(el.tagName==='A'){const href=el.getAttribute('href');if(href){const f=fromHref(href);if(f)add(f);}}el.childNodes&&el.childNodes.forEach(visit);if(el.shadowRoot){el.shadowRoot.childNodes.forEach(visit);}}};visit(document.documentElement);document.querySelectorAll('script[type="application/json"],script').forEach(sc=>{const t=sc.textContent||'';const re=/\"name\"\s*:\s*\"([^\"]{1,240})\"/g;let m;while((m=re.exec(t)))add(m[1]);});const out=[...S].sort((a,b)=>a.localeCompare(b,undefined,{numeric:true,sensitivity:'base'})).join('\n');const copy=async txt=>{try{await navigator.clipboard.writeText(txt);return true;}catch{try{const ta=document.createElement('textarea');ta.value=txt;ta.style.position='fixed';ta.style.left='-9999px';document.body.appendChild(ta);ta.select();document.execCommand('copy');ta.remove();return true;}catch{return false;}}};(async()=>{if(!out){alert('No filenames with known extensions were found.');return;}const ok=await copy(out);alert(ok?`Copied ${S.size} filename${S.size===1?'':'s'} to clipboard.`:'Copy failed.');})();})();
