### **File: `README.md` (Updated)**


# Epic RPG Adventure - Developer Guide

## 📋 Overview
Game RPG berbasis teks command-driven menggunakan Python dan CustomTkinter. Dibangun dengan arsitektur modular untuk kemudahan kolaborasi dan pengembangan.

## 🏗️ Project Structure
```
rpg_game/
├── main.py                    # Entry point
├── core/
│   ├── __init__.py
│   ├── game_engine.py         # Game logic utama
│   ├── command_handler.py     # Handler semua command
│   └── player.py             # Class Player
├── commands/
│   ├── __init__.py
│   ├── basic_commands.py      # /start, /status, dll
│   ├── battle_commands.py     # /attack, /heal, /cast
│   ├── inventory_commands.py  # /inventory, /equip, /use
│   ├── shop_commands.py       # /shop, /buy, /sell
│   ├── crafting_commands.py   # /craft
│   ├── quest_commands.py      # /quest
│   ├── save_commands.py       # /save, /load
│   ├── job_commands.py        # /job, /work, /jobstatus
│   ├── upgrade_commands.py    # /upgrade
│   ├── fusion_commands.py     # /fusion
│   └── cheat_commands.py      # /give, /set, /god
├── data/
│   ├── __init__.py
│   ├── game_data.py          # MONSTERS, ITEMS, dll
│   └── achievements.py       # Achievement definitions
├── ui/
│   ├── __init__.py
│   └── ui_manager.py         # GUI
└── utils/
    ├── __init__.py
    └── helpers.py            # Fungsi bantu
```

## 🚀 Getting Started
1. Pastikan Python 3.7+ terinstall
2. Install dependencies: `pip install customtkinter`
3. Jalankan: `python main.py`

## 🧩 Adding New Commands

### Step 1: Buat file command baru
```python
# commands/new_feature_commands.py
def register_new_feature_commands(handler):
    def my_new_command(arg):
        if not handler.game.player:
            return "❌ You need to start a game first."
        # Logic here
        return "✅ New feature command executed!"
    
    # Register command dengan aliases dan kategori
    handler.register_command('/mycommand', my_new_command, aliases=['/mc'], category='newfeature')
```

### Step 2: Import dan register di `ui/ui_manager.py`
```python
from commands.new_feature_commands import register_new_feature_commands
# Di __init__():
register_new_feature_commands(self.command_handler)
```

### Step 3: Tambahkan ke help jika perlu
Edit `/help` command di `basic_commands.py` untuk menampilkan command baru.

## 🎮 New Features

### 1. Job System
Sistem kerja yang memungkinkan pemain untuk meningkatkan skill dan mendapatkan gold.

#### Commands:
- `/job` - Lihat job yang tersedia
- `/job lumberjack` - Mulai bekerja sebagai lumberjack
- `/job miner` - Mulai bekerja sebagai miner
- `/job farmer` - Mulai bekerja sebagai farmer
- `/work` - Lakukan pekerjaan berdasarkan job yang dipilih
- `/jobstatus` - Lihat status dan level job saat ini

#### Job Details:
- **Lumberjack**: Memotong pohon untuk mendapatkan wood dan material lainnya. Meningkatkan attack secara perlahan.
- **Miner**: Menambang ore untuk mendapatkan material langka. Meningkatkan defense secara perlahan.
- **Farmer**: Menanam dan memanen tanaman. Meningkatkan max HP secara perlahan.

#### Job Levels:
- Setiap job memiliki sistem level XP sendiri
- Semakin tinggi level, semakin besar reward yang didapat
- Bonus stat unik untuk setiap job

### 2. Upgrade System
Sistem untuk meningkatkan kekuatan item senjata dan armor.

#### Commands:
- `/upgrade list` - Lihat item yang bisa diupgrade
- `/upgrade <item_name>` - Upgrade item tertentu (membutuhkan material dan gold)

#### Upgrade Requirements:
- Material spesifik tergantung item
- Gold sebagai biaya upgrade
- Stat meningkat saat upgrade

### 3. Use Command
Command baru untuk equip item secara langsung.

#### Commands:
- `/use <item_name>` - Equip weapon, armor, atau accessory
- `/equip <item_name>` - Alternatif command untuk equip

### 4. Fusion System
Sistem untuk menggabungkan dua item menjadi satu item baru secara acak, dengan kemungkinan hasil yang lebih baik atau lebih buruk tergantung pada `rarity` dan resep fusion.

#### Commands:
- `/fusion <item1> <item2>` - Gabungkan dua item menjadi satu.

#### Fusion Details:
- Membutuhkan resep (`FUSION_RECIPES`) yang didefinisikan di `data/game_data.py`.
- Hasil fusion ditentukan secara acak berdasarkan `chance` dalam resep.
- Item consumable tidak bisa difusion.
- Item dengan `durability` bisa digunakan sebagai input fusion.

### 5. Durability System
Item senjata dan armor kini memiliki `durability` yang menurun saat digunakan dalam pertempuran.

#### Details:
- Item dengan field `durability` dan `durability_max` di `data/game_data.py`.
- Stat item (atk/defense) berkurang seiring dengan penurunan `durability`.
- Jika `durability` mencapai 0, item akan rusak dan otomatis dilepas dari equipment slot.
- Fungsi `damage_equipped_item` di `Player` mengurangi `durability`.

### 6. New Item Rarities
Dua rarity baru telah ditambahkan untuk membuat progresi late-game lebih menarik.

#### Rarities:
- **Transcendent** (warna: Biru Tua / Indigo)
- **Cosmic** (warna: Ungu Gelap / Navy)

#### Details:
- Ditambahkan ke `game_data.py` untuk monster, item, crafting, dan quest.
- Ditambahkan ke `ui_manager.py` untuk sistem pewarnaan otomatis di UI.

## 📊 Data Structures

### Monster Format
```python
{
    "name": "Goblin",
    "hp": 30,
    "atk": 8,
    "exp": 20,
    "gold": 10,
    "min_level": 1,
    "max_level": 3,
    "rarity": "common",
    "drops": [{"item": "Iron Ore", "chance": 0.3}],
    "boss": True  # Opsional, default False
}
```

### Item Format
```python
{
    "name": "Heal Potion",
    "type": "consumable",  # consumable, weapon, armor, material, accessory
    "effect": "heal_30",   # Hanya untuk consumable
    "price": 25,
    "rarity": "common",
    "atk": 10,             # Hanya untuk weapon
    "defense": 5,          # Hanya untuk armor
    "durability": 100,     # Opsional, untuk sistem durability
    "durability_max": 100, # Opsional, untuk sistem durability
    "required_job_level": 3 # Opsional, level job minimum untuk menggunakan item
}
```

### Quest Format
```python
{
    "id": "kill_5_monsters",
    "name": "Slay 5 Monsters",
    "target": "any",       # any, specific_monster_name, level, gold, boss
    "goal": 5,
    "reward": {"gold": 100, "item": "Heal Potion"}
}
```

### Fusion Recipe Format
```python
{
    "input": ["Iron Ore", "Iron Ore"],  # Nama item input (harus sesuai)
    "output_pool": [                    # Pool hasil dengan probabilitas
        {"item": "Steel Ingot", "chance": 0.6},
        {"item": "Rusty Sword", "chance": 0.3},
        {"item": "Iron Sword (Dura)", "chance": 0.1}
    ]
}
```

## 🔧 Core Components

### GameEngine
- Menyimpan state permainan (`self.player`, `self.current_enemy`, dll)
- Handle battle mechanics
- Save/load game
- Achievement checking

### CommandHandler
- Mengelola semua command
- Handle aliases
- Route command ke fungsi yang benar
- Handle categorization untuk help system

### Player
- Menyimpan semua data karakter
- Handle stat calculations
- Equipment management
- Job system integration
- Upgrade system integration
- Fusion & Durability integration

## 🧪 Testing Commands
Command yang bisa digunakan untuk testing:
- `/start Hero` - Mulai game
- `/job lumberjack` - Pilih job lumberjack
- `/work` - Lakukan pekerjaan
- `/upgrade list` - Lihat item yang bisa diupgrade
- `/use Iron Sword` - Equip senjata
- `/fusion Iron Ore Iron Ore` - Gabungkan dua item
- `/give Heal Potion 10` - Tambah item (cheat)
- `/set level 10` - Set level (cheat)
- `/god` - Mode dewa (cheat)

## 📝 Notes Penting
- Jangan hardcode values di command handler, gunakan `data.game_data.py`
- Pastikan semua command return string error yang informatif
- Gunakan emoji konsisten untuk feedback visual
- Simpan progress ke file JSON untuk persistensi
- Tambahkan kategori saat register command untuk sistem help yang lebih baik
- Pastikan UI (`ui_manager.py`) diupdate jika ada rarity atau command baru yang mempengaruhi tampilan.

## 🤝 Contributing
1. Buat feature branch: `git checkout -b feature/nama-fitur`
2. Tambahkan command di folder `commands/`
3. Import di `ui_manager.py`
4. Update `basic_commands.py` jika perlu menambahkan help
5. Update `data/game_data.py` jika menambahkan data baru (item, monster, quest, dll)
6. Update `README.md` jika menambahkan fitur baru
7. Commit dan push
8. Buat Pull Request
