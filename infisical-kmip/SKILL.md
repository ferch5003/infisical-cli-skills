---
name: infisical-kmip
description: KMIP server operations for key management
Triggers: infisical kmip, kmip server, key management
Commands: server start, keys list, encrypt, decrypt
Examples:
  infisical kmip server start --port=5696
  infisical kmip keys list
  infisical kmip encrypt --key-id=mykey --plaintext=secret
---