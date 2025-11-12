# 🎯 STAGE 3 IMPLEMENTATION PLAN — Lightway + HTTPS + Persistence

## Objective

Make RMADA production-ready with:
1. **Lightway VPN** — Efficient VPN server + portable builds
2. **HTTPS/TLS** — Secure transport (self-signed + Let's Encrypt ready)
3. **Persistence** — SQLite database for telemetry
4. **Mobile Support** — WireGuard fallback + Linux gateway guidance
5. **Portable Deployment** — Docker images + tarballs for any machine (including ARM64)

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    RMADA Stage 3 Architecture               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Mobile Client (iOS/Android)                               │
│  ├─ Option 1: WireGuard (native app)                       │
│  └─ Option 2: Connect via Linux gateway (Lightway relay)   │
│                                                             │
│  Device/Linux Client                                       │
│  ├─ Lightway Client ← → Lightway Server (port 1024)        │
│  └─ Or WireGuard ← → WireGuard Server (port 51820)         │
│                                                             │
│  RMADA Server (Linux/Docker)                               │
│  ├─ Node.js (port 8080, HTTPS 8443)                        │
│  ├─ Lightway Server (port 1024)                            │
│  ├─ WireGuard Server (port 51820, optional fallback)       │
│  └─ SQLite Database (telemetry, users, devices)            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Tasks Breakdown

### 1. Lightway VPN Server (Rust)
- [x] Build Lightway server binary from source (already in repo)
- [ ] Create Earthly targets for cross-compilation (x86_64, ARM64)
- [ ] Package as portable tarball
- [ ] Create startup script (lightway-startup.sh)
- [ ] Integrate with server.js (device auth)

### 2. HTTPS/TLS Support
- [ ] Add Node.js native HTTPS support
- [ ] Generate self-signed certificates (development)
- [ ] Document Let's Encrypt integration
- [ ] Update frontend to support HTTPS
- [ ] Add HSTS + security headers

### 3. SQLite Persistence
- [ ] Create database schema
- [ ] Add sqlite3 package to Node.js
- [ ] Create migrations
- [ ] Update server.js to use DB (users, devices, telemetry)
- [ ] Create backup scripts

### 4. Mobile Support
- [ ] Document WireGuard fallback for iOS/Android
- [ ] Create Linux gateway setup guide
- [ ] Provide QR codes for mobile device setup
- [ ] Document Lightway limitations on mobile

### 5. Portable Deployment
- [ ] Create Earthly targets:
  - +lightway-image (x86_64)
  - +lightway-arm64 (ARM64 for Raspberry Pi)
  - +lightway-tarball (extractable anywhere)
  - +complete-stage3-image (all-in-one)
- [ ] Test cross-platform builds
- [ ] Document deployment options

### 6. Documentation
- [ ] README-STAGE3.md — Quick start
- [ ] LIGHTWAY-SETUP.md — VPN setup guide
- [ ] HTTPS-SETUP.md — HTTPS configuration
- [ ] DATABASE-SETUP.md — SQLite guide
- [ ] MOBILE-GUIDE.md — Mobile client setup
- [ ] DEPLOYMENT-PRODUCTION.md — Production checklist
- [ ] ARCHITECTURE.md — Full system design

### 7. Testing
- [ ] End-to-end test with Lightway + HTTPS
- [ ] Database persistence tests
- [ ] Mobile client tests (WireGuard)
- [ ] Failover tests (WireGuard fallback)

---

## Priority Order

### Phase 1 (Foundation) — 2 hours
1. Add HTTPS support to Node.js
2. Create SQLite integration
3. Basic database schema

### Phase 2 (VPN) — 3 hours
1. Lightway startup script
2. Earthly targets for Lightway
3. Tarball packaging

### Phase 3 (Mobile) — 2 hours
1. WireGuard fallback integration
2. Mobile guidance documentation
3. Linux gateway setup

### Phase 4 (Documentation & Testing) — 2 hours
1. Comprehensive documentation
2. E2E test suite
3. Deployment guides

---

## Key Files to Create/Modify

### Create
- `lightway-startup.sh` — Lightway server launcher
- `database-schema.sql` — SQLite schema
- `database-init.js` — Database initialization
- `https-config.js` — HTTPS/TLS setup
- `README-STAGE3.md`
- `LIGHTWAY-SETUP.md`
- `HTTPS-SETUP.md`
- `DATABASE-SETUP.md`
- `MOBILE-GUIDE.md`
- `DEPLOYMENT-PRODUCTION.md`
- `ARCHITECTURE.md`
- `test-stage3-e2e.sh`

### Modify
- `Earthfile` — Add Lightway + Stage 3 targets
- `server.js` — Add HTTPS + database + Lightway integration
- `package.json` — Add sqlite3 dependency
- `docker-compose.yml` — Add database volume + HTTPS ports
- `app.js` — Update frontend for HTTPS

---

## Success Criteria

✅ Lightway VPN server running in Docker  
✅ HTTPS/TLS working (self-signed)  
✅ SQLite database persisting telemetry  
✅ Mobile devices connecting via WireGuard  
✅ Linux clients connecting via Lightway  
✅ Portable tarballs for any Linux system  
✅ ARM64 builds for Raspberry Pi  
✅ Complete documentation  
✅ E2E tests passing  

---

## Timeline

- **Phase 1**: ~2 hours (HTTPS + SQLite basics)
- **Phase 2**: ~3 hours (Lightway + portable builds)
- **Phase 3**: ~2 hours (Mobile + gateway setup)
- **Phase 4**: ~2 hours (Docs + tests)

**Total**: ~9 hours → Production-ready Stage 3

---

## Risk Mitigation

| Risk | Mitigation |
|------|-----------|
| Lightway build complexity | Use existing lightway-main repo |
| Cross-compilation failures | Test on multiple architectures |
| Mobile app unavailability | WireGuard fallback (native apps exist) |
| Database migration issues | Backward-compatible schema design |
| HTTPS certificate management | Support both self-signed + Let's Encrypt |

---

**Ready to start! 🚀**
