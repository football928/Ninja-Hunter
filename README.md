// NinjaHunter - TikTok Gallery (React single-file)
// Default export: React component
// Tailwind CSS assumed in project (no import needed)
// This file includes:
// 1) A React component that displays a grid of TikTok embeds (client-side)
// 2) A tiny helper to dynamically load TikTok embed script
// 3) Example Node/Express server code (commented) that proxies TikTok oEmbed (recommended to avoid CORS and rate limits)

import React, { useEffect, useRef } from "react";

// ---------- CONFIG ----------
// Replace these example TikTok URLs with your own video URLs or IDs.
const SAMPLE_TIKTOK_URLS = [
https://www.tiktok.com/@sasaabukarma0?is_from_webapp=1&sender_device=pc
[https://www.tiktok.com/@sasaabukarma0?is_from_webapp=1&sender_device=pc
](https://www.tiktok.com/@sasaabukarma0?is_from_webapp=1&sender_device=pc)  // add more urls here
];

// ---------- HELPERS ----------
function loadTikTokEmbedScript() {
  // ensure script is only injected once
  if (typeof window === "undefined") return;
  if ((window as any).__tiktok_embed_loaded) return;
  const s = document.createElement("script");
  s.src = "https://www.tiktok.com/embed.js";
  s.async = true;
  s.onload = () => {
    (window as any).__tiktok_embed_loaded = true;
  };
  document.body.appendChild(s);
}

function TikTokEmbed({ url }: { url: string }) {
  const ref = useRef<HTMLDivElement | null>(null);

  useEffect(() => {
    loadTikTokEmbedScript();
    // When embed script is ready it scans the DOM for blockquote.tiktok-embed elements and initialises them.
    // If the script was already loaded, we call its parse function if available.
    const tryParse = () => {
      const t = (window as any).tiktok;
      if (t && typeof t.parse === "function") {
        try {
          t.parse(ref.current);
        } catch (e) {
          // ignore parse errors
        }
      } else {
        // some versions put parse on window.__tiktok or window.parseTikTok
        const globalParse = (window as any).parseTikTok || (window as any).__tiktokParse;
        if (typeof globalParse === "function") {
          try { globalParse(ref.current); } catch (e) {}
        }
      }
    };

    // attempt parse after small delay to allow script to run
    const id = setTimeout(tryParse, 600);
    return () => clearTimeout(id);
  }, [url]);

  // TikTok embed markup: blockquote with cite attribute and a placeholder section
  return (
    <div ref={ref} className="w-full">
      <blockquote
        className="tiktok-embed"
        cite={url}
        data-video-url={url}
        style={{ maxWidth: 552, minWidth: 325 }}
      >
        <section>Loading TikTok...</section>
      </blockquote>
    </div>
  );
}

export default function TikTokGallery() {
  // You can replace SAMPLE_TIKTOK_URLS with your own fetched list (API, Google Sheet, DB)
  const videos = SAMPLE_TIKTOK_URLS;

  return (
    <div className="min-h-screen bg-gray-50 p-6">
      <div className="max-w-5xl mx-auto">
        <header className="flex items-center justify-between mb-6">
          <div>
            <h1 className="text-2xl font-bold">NinjaHunter - TikTok Gallery</h1>
            <p className="text-sm text-gray-600">مكتبة فيديوهات TikTok المرتبطة بقناة @NinjaHunter-l9x</p>
          </div>
          <div className="text-right">
            <a
              href="https://www.tiktok.com/@sasaabukarma0?is_from_webapp=1&sender_device=pc
              target="_blank"
              rel="noreferrer"
              className="px-4 py-2 rounded-lg border text-sm"
            >
              افتح حساب التيك توك
            </a>
          </div>
        </header>

        <main>
          <div className="grid grid-cols-1 sm:grid-cols-2 gap-6">
            {videos.map((url, i) => (
              <article key={i} className="bg-white p-4 rounded-2xl shadow-sm">
                <TikTokEmbed url={url} />
                <div className="mt-3 text-sm text-gray-700">
                  <div>وصف قصير للفيديو — استبدله بمحتوى ديناميكي</div>
                  <div className="mt-2 flex items-center justify-between">
                    <button className="px-3 py-1 rounded-lg border text-xs">حفظ</button>
                    <a
                      href={url}
                      target="_blank"
                      rel="noreferrer"
                      className="text-xs underline"
                    >
                      فتح في تيك توك
                    </a>
                  </div>
                </div>
              </article>
            ))}
          </div>
        </main>

        <footer className="mt-8 text-center text-sm text-gray-500">© NinjaHunter - استخدم الفيديوهات المتاحة للعرض فقط ولا تنتهك حقوق الملكية</footer>
      </div>
    </div>
  );
}

// -------------------------
// OPTIONAL: Simple Node/Express proxy for TikTok oEmbed (server-side). Save as server.js and run with Node.
// This helps avoid CORS problems when calling TikTok's oEmbed endpoint from the browser.
// -------------------------
/*
const express = require('express');
const fetch = require('node-fetch');
const app = express();
const PORT = process.env.PORT || 3001;

app.get('/api/oembed', async (req, res) => {
  const { url } = req.query;
  if (!url) return res.status(400).json({ error: 'missing url' });
  try {
    const tgt = `https://www.tiktok.com/oembed?url=${encodeURIComponent(url)}`;
    const r = await fetch(tgt, { headers: { 'User-Agent': 'NinjaHunter/1.0' } });
    if (!r.ok) return res.status(502).json({ error: 'bad response from tiktok' });
    const json = await r.json();
    return res.json(json);
  } catch (err) {
    return res.status(500).json({ error: 'server error', details: String(err) });
  }
});

app.listen(PORT, () => console.log('TikTok proxy running on', PORT));
*/

// -------------------------
// DEPLOY NOTES & SUGGESTIONS (short):
// - Frontend: deploy on Vercel / Netlify as a React app.
// - If you use the oEmbed proxy, host it on the same domain or on a small server (Heroku, Render, Fly.io).
// - Add SEO: title, meta description, open graph tags, and structured data for video if needed.
// - Legal: ensure you have the right to embed or re-publish videos. Prefer embedding official TikTok content rather than hosting copies.
// - Improvements: lazy-load embeds, user uploads, search, categorization (by game/weapon/guide), add comments/likes using your own DB.

// -------------------------
// END FILE
