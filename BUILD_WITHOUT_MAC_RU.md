# Сборка DefaultVPN для iPhone без собственного Mac

В этой версии исправлено применение **site-based split tunneling** к IPv4-only AWG/WireGuard-конфигам с `AllowedIPs = 0.0.0.0/0`.

Приложение уже содержит экран **Settings → Split tunneling → Site-based split tunneling**. Выберите режим **«Addresses from the list should not be accessed via VPN»** и добавляйте IP/CIDR исключения. Они будут передаваться в iOS Network Extension как исключённые маршруты, а основной AWG-конфиг может оставаться коротким с `AllowedIPs = 0.0.0.0/0`.

## Что изменено

1. `client/vpnconnection.cpp` — IPv4 default route `0.0.0.0/0` теперь достаточно, чтобы включить site-based split tunneling; `::/0` больше не обязателен.
2. `client/platforms/ios/PacketTunnelProvider+WireGuard.swift` — iOS применяет `splitTunnelSites`/`excludeIPs` и для IPv4-only профилей.
3. `.github/workflows/ios-ipa-manual.yml` — ручная сборка подписанного `.ipa` на GitHub Actions macOS runner.

## Сборка через GitHub Actions

1. Создайте приватный GitHub-репозиторий и загрузите туда содержимое этой папки.
2. Нужна Apple-подпись, которая разрешает Network Extension. Для приложения и Network Extension используются отдельные provisioning profiles.
3. В GitHub: **Settings → Secrets and variables → Actions** добавьте обязательные secrets:
   - `IOS_SIGNING_CERT_BASE64` — `.p12` сертификат в Base64.
   - `IOS_SIGNING_CERT_PASSWORD` — пароль `.p12`.
   - `IOS_TRUST_CERT_BASE64` — Apple trust/intermediate certificate в Base64, который ожидает исходный `deploy/build_ios.sh`.
   - `IOS_APP_PROVISIONING_PROFILE` — provisioning profile основного приложения в Base64.
   - `IOS_NE_PROVISIONING_PROFILE` — provisioning profile Network Extension в Base64.
4. При необходимости добавьте проектные secrets, используемые исходным DefaultVPN: `PROD_AGW_PUBLIC_KEY`, `PROD_S3_ENDPOINT`, `FALLBACK_S3_ENDPOINT`, `DEV_AGW_PUBLIC_KEY`, `DEV_AGW_ENDPOINT`, `DEV_S3_ENDPOINT`, `FREE_V2_ENDPOINT`, `PREM_V1_ENDPOINT`.
5. Откройте **Actions → Build signed iOS IPA → Run workflow**.
6. После сборки скачайте artifact **DefaultVPN-iOS-split-exclusions**. В нём будет `DefaultVPN-split-exclusions.ipa` и информация о подписи.

## Важно

GitHub Actions предоставляет Mac для сборки, но не заменяет Apple code signing. Для VPN Network Extension нужен корректный entitlement/provisioning profile. Без подписывающих данных workflow остановится до сборки и покажет имя отсутствующего secret.

Для вашей схемы рекомендуется оставить в AWG-конфиге:

```ini
AllowedIPs = 0.0.0.0/0
```

а нужные российские IP/CIDR добавлять в **Split tunneling → Addresses from the list should not be accessed via VPN**. Это не раздувает сам AWG-ключ сотнями `AllowedIPs`.
