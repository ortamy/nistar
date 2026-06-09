# 📁 Структура репозитория Nistar

nistar-system/
├── README.md
├── LICENSE
├── ARCHITECTURE.md              # Полная архитектура системы
├── THREAT_MODEL.md              # Модель угроз и допущения
├── DEPLOYMENT.md                # Руководство по развёртыванию "с нуля"
│
├── nistar-os/                   # Подсистема: Операционная среда
│   ├── README.md
│   ├── build/
│   │   ├── nistar-os.iso.shasum # Контрольная сумма образа
│   │   └── Dockerfile           # Сборка кастомного Tails
│   ├── config/
│   │   ├── persistence.conf     # Разделы для сохранения
│   │   ├── torrc               # Конфиг Tor
│   │   └── i2p.conf            # Конфиг I2P-роутера
│   ├── scripts/
│   │   ├── nistar-lock.sh      # Мгновенная блокировка/зачистка RAM
│   │   ├── nistar-wipe.sh      # Криптографическая зачистка носителей
│   │   └── amnesia-check.sh    # Проверка отсутствия утечек на диск
│   └── docs/
│       ├── BOOT_INTEGRITY.md   # Проверка целостности загрузки
│       └── USB_CREATION.md     # Создание загрузочного носителя
│
├── nistar-net/                  # Подсистема: Сетевая маршрутизация
│   ├── README.md
│   ├── whonix/                  # Шлюзы на базе Whonix
│   │   ├── gateway/
│   │   │   ├── Dockerfile
│   │   │   └── entrypoint.sh
│   │   └── workstation/
│   │       ├── Dockerfile
│   │       └── entrypoint.sh
│   ├── vpn/                    # Собственный VPN
│   │   ├── shadowsocks/
│   │   │   ├── server-config.json
│   │   │   └── deploy.sh       # Авторазвёртывание на VPS
│   │   └── wireguard/
│   │       ├── server.conf
│   │       └── client.conf.template
│   ├── bridges/                # Мосты Tor
│   │   ├── obfs4-bridges.txt   # Актуальные мосты
│   │   └── snowflake-setup.md
│   ├── scripts/
│   │   ├── nistar-connect.sh   # Автоматическое подключение (Tor→VPN→I2P)
│   │   ├── killswitch.sh       # Обрыв соединений при утечке
│   │   └── dpi-bypass.py       # Обход DPI (фрагментация, рандомизация)
│   └── docs/
│       ├── DPI_BYPASS.md       # Обход глубокой инспекции пакетов
│       └── OWN_VPN.md          # Собственный VPN за XMR
│
├── nistar-comms/               # Подсистема: Коммуникации
│   ├── README.md
│   ├── matrix/
│   │   ├── synapse-deploy.yml  # Развёртывание своего сервера Matrix
│   │   └── element-config.json # Конфиг клиента без утечек
│   ├── simplex/
│   │   └── relay-deploy.md     # Поднятие своего релея SimpleX
│   ├── email/
│   │   ├── proton-bridge.sh    # Использование ProtonMail через Tor
│   │   └── onionmail.md        # Почта только в .onion
│   ├── scripts/
│   │   ├── exif-strip.sh       # Пакетная очистка метаданных
│   │   ├── metadata-scrub.py   # Глубокая зачистка метаданных файлов
│   │   └── temp-sms.sh         # Аренда виртуального номера
│   └── docs/
│       ├── NO_PHONE_REG.md     # Регистрация без номера телефона
│       └── SIGNAL_VOIP.md      # Signal без привязки к SIM
│
├── nistar-pay/                 # Подсистема: Финансы
│   ├── README.md
│   ├── xmr/
│   │   ├── monero-wallet.md    # Кошелёк Monero (Feather Wallet + Tor)
│   │   ├── xmr-bisq.md         # P2P-обмен XMR через Bisq
│   │   └── haveno-setup.md     # Покупка/продажа за наличные (Haveno)
│   ├── swaps/
│   │   ├── trocador.md         # Мгновенный своп BTC→XMR
│   │   └── majesticbank.md     # Обмен через Telegram-ботов
│   ├── cards/
│   │   └── virtual-cards.md    # Виртуальные карты без KYC
│   ├── scripts/
│   │   └── xmr-check.sh        # Проверка баланса через CLI
│   └── docs/
│       ├── CASH_TO_XMR.md      # Наличные → Monero (офлайн, P2P)
│       ├── XMR_TO_CASH.md      # Monero → наличные
│       └── BANKS_NO_BIOMETRY.md # Банки РФ без биометрии
│
├── nistar-id/                  # Подсистема: Цифровые идентичности
│   ├── README.md
│   ├── profiles/               # Шаблоны изолированных профилей
│   │   ├── public.json         # Публичный профиль
│   │   ├── professional.json   # Профессиональный профиль
│   │   └── shadow.json         # Теневой профиль
│   ├── scripts/
│   │   ├── id-switch.sh        # Переключение между профилями
│   │   └── generate-id.py      # Генератор виртуальной личности
│   └── docs/
│       ├── COMPARTMENTATION.md # Полное разделение идентичностей
│       └── BURNED.md           # Протокол при компрометации
│
├── scenarios/                  # Готовые операционные сценарии
│   ├── scenario-zero.md        # Развёртывание "с нуля" (полный стек)
│   ├── scenario-device-seized.md # Действия при изъятии устройства
│   ├── scenario-team-comms.md  # Анонимная коммуникация в группе
│   ├── scenario-freelance.md   # Получение оплаты без паспорта
│   └── scenario-cross-border.md # Пересечение границы с данными
│
├── specs/                      # Спецификации и стандарты
│   ├── NISTAR-OS-1.0.md        # Спецификация Nistar OS
│   ├── NISTAR-NET-1.0.md       # Спецификация сетевого стека
│   └── NISTAR-COMMS-1.0.md     # Спецификация коммуникаций
│
└── tools/                      # Вспомогательные инструменты
    ├── usb-imager.md           # Верифицированные программы для записи USB
    ├── hash-verifier.sh        # Скрипт проверки хешей
    └── hardware-recs.md        # Рекомендованное железо
