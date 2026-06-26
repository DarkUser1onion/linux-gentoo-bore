# Кастомное ядро Linux 7.0.11 с патчами Gentoo и планировщиком BORE (CachyOS)

Данная сборка основана на ванильном ядре Linux 7.0.11 (ветка stable).
Поверх него наложены:
- 11 патчей от Gentoo (genpatches) для совместимости с Portage и безопасности.
- Патч планировщика BORE (Burst-Oriented Response Enhancer) из репозитория CachyOS.

## Установка
```bash
git clone https://github.com/DarkUser1onion/linux-gentoo-bore
cd linux-gentoo-bore
make olddefconfig
make -j$(nproc)
sudo make modules_install install
```

## Список включённых патчей Gentoo (все, кроме инкрементов 1000-1010 и BMQ/PDS):

- 1510_fs-enable-link-security-restrictions-by-default.patch
- 1605_crypto-nx-fix-nx-crypto-ctx-exit-arg.patch
- 1700_sparc-address-warray-bound-warnings.patch
- 1730_parisc-Disable-prctl.patch
- 2000_BT-Check-key-sizes-only-if-Secure-Simple-Pairing-enabled.patch
- 2901_permit-menuconfig-sorting.patch
- 2902_Replace-CONST-CAST-with-const-cast.patch
- 2990_libbpf-v2-workaround-Wmaybe-uninitialized-false-pos.patch
- 2991_libbpf_add_WERROR_option.patch
- 3000_Support-printing-firmware-info.patch
- 4567_distro-Gentoo-Kconfig.patch

## Патч CachyOS:
- 0001-bore.patch (планировщик BORE, версия без зависимостей от CONFIG_CACHY)

# ВАЖНЫЕ ПРИМЕЧАНИЯ:
- Патчи nvidia не использовались, так как собирал ядро под AMD карты
- Патчи инкрементального обновления (1000_linux-7.0.1.patch ... 1010_linux-7.0.11.patch) не применялись, так в ядре 7.0.11 они уже были.
- Патчи BMQ/PDS (5020_* и 5021_*) исключены из-за конфликта с BORE.
- Дополнительные патчи CachyOS (cgroup-vram, hardened, aufs и др.) не использованы.

## Конфиг максимально сильно, насколько возможно, оптимизирован под мое железо

### (Hardware Configuration)

Система, на которой собран и протестирован данный конфиг ядра:

| Компонент | Модель / Характеристики |
|-----------|-------------------------|
| **Процессор** | Intel Xeon E5-2697A v4 (Broadwell-EP) |
| **Материнская плата** | Machinist MR9A Pro Max v2.1 (чипсет Intel C612 / B85) |
| **Видеокарта** | AMD Radeon RX 6800 XT (Navi 21) |
| **Оперативная память** | Hynix DDR4, ECC |
| **Диски** | • NVMe: ARDOR GAMING m.2 NVMe<br>• SATA SSD: 512 ГБ <br>• SATA HDD: 750 ГБ WD Blue, 1 ТБ WD Blue , 1 ТБ WD Blue|
| **Файловая система** | XFS (корневая), остальные разделы — XFS/ext4/vfat |
| **Сетевой интерфейс** | Realtek RTL8111/8168 PCI Express Gigabit Ethernet |
| **USB / аудио** | USB-гарнитуры и микрофоны через драйвер `snd-usb-audio` |

### Планирование и отзывчивость
- Планировщик BORE (CONFIG_SCHED_BORE=y).
- Частота таймера 1000 Гц (CONFIG_HZ=1000).
- Полная вытесняемость (CONFIG_PREEMPT=y).
- NUMA-осведомлённый планировщик (CONFIG_NUMA_BALANCING=y).
- SMT (Hyper-Threading) - полностью задействованы все потоки.

### Производительность и низкие задержки
- Отключены все митигации Spectre/Meltdown.
- Отключены управление питанием PCIe ASPM (pcie_aspm=off) и PCIe AER (pci=noaer).
- Ограничение C-состояний процессора (processor.max_cstate=1, intel_idle.max_cstate=0).
- Отключён сторожевой таймер (nowatchdog).

### Графика (AMD RX 6800 XT)
- Драйвер amdgpu собран как модуль (гибкость, меньший размер ядра).
- Отключён отложенный захват консоли (fbcon=nodefer, CONFIG_FRAMEBUFFER_CONSOLE_DEFERRED_TAKEOVER=n).
- Отключено динамическое управление питанием GPU (amdgpu.runpm=0).
- Отключена поддержка HSA/AMDKFD (CONFIG_HSA_AMD=n).

### Вычищены все неиспользуемые драйверы и подсистемы
- Полностью удалены: FireWire (IEEE 1394), PCMCIA/CardBus, Parallel port, ISDN, FDDI, ARCnet, ATM, CAN, MCTP, SLIP, EQL.
- Отключены старые IDE и лишние SATA/PATA контроллеры – оставлен только AHCI и Intel ESB/ICH/PIIX для совместимости.
- Отключены экзотические шины: I3C, HSI, MTD, 1-wire, IndustryPack, MCB, SoundWire.
- Отключены сетевые технологии: Distributed Switch Architecture (DSA), vDPA, VXLAN, GENEVE, GTP, PFCP, AMT, MACsec, PPP.
- Отключены Android-специфичные модули: Binder, Binderfs.
- Отключены виртуализационные фишки не используемые: VFIO, vDPA, Hyper-V guest.
- Отключены промышленные и отладочные драйверы: Comedi, GPIB, IIO, FPGA, TEE, Counter, HTE.
- Отключена поддержка NVDIMM, DAX, Android, Surface, Chrome, Mellanox, Goldfish.
- Выключены все Staging драйверы (экспериментальные/нестабильные).

### Компиляция и оптимизация кода
- Архитектурная оптимизация: флаг -march=native – ядро собирается строго под инструкции Xeon Broadwell (AVX2, BMI, FMA, ...).
- Уровень оптимизации -O3 (вместо стандартного -O2) – агрессивная векторизация, разворачивание циклов, инлайнинг.
- Дополнительный флаг -pipe – ускоряет компиляцию.
- Отключены отладочные опции: DEBUG_KERNEL, KPROBES, LOCK_DEBUG, DEBUG_INFO, KALLSYMS (кроме необходимых зависимостей), TRACERS.
- Включена smaller-sized data structures – экономия памяти.
- Link Time Optimization (LTO) – отключено (совместимость, размер).

### Файловая система и диски
- NVMe – драйвер включён, поддержка APST (управление питанием) отключен.
- Планировщик ввода-вывода: none (для NVMe) / bfq (для HDD).
- Отключены все неиспользуемые файловые системы (FUSE, CIFS, NFS, AFS, etc.) – оставлены только XFS, EXT4 (модуль), btrf (модуль), proc, sysfs, tmpfs, devtmpfs, debugfs.
- XFS (с поддержкой online-проверки и восстановления метаданных — `CONFIG_XFS_ONLINE_SCRUB/REPAIR`).
- **Полностью отключены**:
  - Btrfs.
  - FSCache
  - FUSE
  - Overlayfs
  - Сетевые файловые системы: NFS (клиент и сервер), CIFS/SMB, Ceph, AFS, Coda, 9P
  - JFS, GFS2, OCFS2, NILFS2, F2FS, zonefs
- **Отключены** все отладочные и тестовые опции (KUnit tests, debugging support) для каждой ФС.

### Сеть
- Оставлен только драйвер Realtek R8169 – все остальные Ethernet-драйверы вырезаны.
- Оставлены минимальные сетевые модули: WireGuard, OpenVPN offload, TUN/TAP, dummy, MAC-VLAN/IP-VLAN (опционально), Virtio (для KVM).
- Вырезаны все прочие сетевые протоколы и фильтрации (L2TP, PPTP, VXLAN, GENEVE, IPVLAN, MACsec, AMT, GTP, PFCP, DSA, CAN, MCTP, и т.д.).
- PHY-драйвер оставлен минимально необходимый.

### Звук
- Оставлен только драйвер Intel HD Audio (snd_hda_intel) для встроенного кодека.
- Выключены USB-аудио, MIDI, профессиональные драйверы.
- Включена быстрая перезагрузка кодека (CONFIG_SND_HDA_HWDEP=y) и предвыделение буфера 2048 байт (CONFIG_SND_HDA_PREALLOC_SIZE=2048) – снижение задержек.

### Энергосбережение и ACPI
- Отключены: Suspend to RAM и Hibernation.
- Отключена Energy Model.
- Оставлен Cpuidle Driver for Intel Processors.
- ACPI оставлен в минимальной конфигурации, без отладочных опций.
