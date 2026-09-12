# RoomBook

ANEW eğitiminde spec'ten teslime geliştirilen toplantı odası rezervasyon API'si.

- [RoomBook uygulama playbook'u](PLAYBOOK.md)
- [Codex çalışma kuralları](AGENTS.md)
- [ANEW workflow](workflows/README.md)
- [Aktif spec'ler](specs/active/README.md)

## Başlangıç

```bash
./scripts/doctor
codex
```

Codex oturumunda `$anew-workflow` skill'ini çağır ve önce bootstrap workflow'unu tamamla.
Bootstrap sırasında dil, framework, test aracı ve `lite`/`strict` çalışma modu seçilecek.
