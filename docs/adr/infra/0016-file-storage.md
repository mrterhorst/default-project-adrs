# ADR-0016: File storage — Cloudflare R2

**Scope:** storage, infra

## Status
Accepted

## Context
De applicatie heeft object storage nodig voor drie typen bestanden: user uploads (afbeeldingen, documenten), server-gegenereerde bestanden (PDFs, exports) en statische assets. De stack draait op Cloudflare Workers.

## Beslissing
Cloudflare R2 wordt gebruikt als object storage. R2 is S3-compatible en heeft geen egress-kosten (download-verkeer is gratis). De toegangspatronen verschillen per bestandstype:

### User uploads → presigned URLs
De Worker genereert een tijdelijke gesigneerde upload-URL; de client uploadt direct naar R2 zonder via de Worker te gaan:

```
client  →  POST /upload/presign  →  Worker  →  R2.createMultipartUpload()
                                  ←  { uploadUrl, key }
client  →  PUT {uploadUrl}  →  R2 (direct)
client  →  POST /upload/complete  →  Worker  →  slaat key op in DB
```

Voordelen: geen Worker-geheugenlimieten, snellere uploads, minder CPU-verbruik op de Worker.

### Server-gegenereerde bestanden → Workers binding
De Worker schrijft gegenereerde bestanden (PDFs, exports) direct naar R2 via de binding:

```typescript
await env.BUCKET.put(`invoices/${id}.pdf`, pdfBuffer, {
  httpMetadata: { contentType: 'application/pdf' },
})
```

Downloads via een Worker-endpoint dat een tijdelijke gesigneerde download-URL genereert — nooit directe publieke toegang tot gegenereerde bestanden.

### Statische assets → R2 public bucket of Cloudflare Pages
Vaste bestanden (fonts, icons, media) worden geserveerd vanuit een publieke R2-bucket of Cloudflare Pages. Geen Workers-tussenlaag nodig.

### Lokale development
Wrangler emuleert R2 lokaal als een map op het bestandssysteem (zie ADR-0011). Geen echte R2-account nodig tijdens development.

### Wrangler-configuratie
```toml
[[r2_buckets]]
binding = "BUCKET"
bucket_name = "mijnproject-uploads"
preview_bucket_name = "mijnproject-uploads-preview"
```

## Consequenties
**Positief:**
- Geen egress-kosten: downloads van R2 zijn gratis, ongeacht volume.
- S3-compatible API: vertrouwde interface, breed ondersteund door libraries.
- Presigned URLs voor uploads omzeilen Worker-limieten en zijn de industriestandaard (zelfde patroon als AWS S3).
- Wrangler-emulatie maakt offline development mogelijk zonder echte R2-kosten.
- Gegenereerde bestanden nooit publiek toegankelijk — altijd via tijdelijke URLs.

**Negatief:**
- Presigned URL flow vereist twee API-aanroepen (presign + complete) in plaats van één directe upload.
- R2 heeft geen ingebouwde beeldoptimalisatie — voor image resizing is Cloudflare Images of een aparte service nodig.

## Alternatieven overwogen
- **AWS S3:** Zelfde interface (S3-compatible), maar hogere egress-kosten en buiten het Cloudflare-ecosysteem. Valt af op kosten en integratie.
- **Cloudflare Images:** Geschikt voor afbeeldingen met automatische resizing, maar beperkt tot afbeeldingsformaten en duurder per transformatie. Kan naast R2 worden ingezet voor specifieke use cases.
- **Supabase Storage:** Gebouwd op S3, maar voegt een extra serviceafhankelijkheid toe. Valt af op complexiteit.
