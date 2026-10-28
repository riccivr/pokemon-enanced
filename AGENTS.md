# AGENTS.md - Pokémon ENANCED Edition

## 1. Project Overview & Context
- **Project Name:** Pokémon ENANCED Edition
- **Engine Base:** `pret/pokefirered` GBA decompilation
- **Repository:** `https://github.com/riccivr/pokemon-enanced`
- **Reference Docs:**
  - Plan: `C:\Users\ricci\.gemini\antigravity\brain\4b6d8221-8c5e-42ec-9d1b-7ef8cde150d9\project_plan.md`
  - Script: `C:\Users\ricci\.gemini\antigravity\brain\4b6d8221-8c5e-42ec-9d1b-7ef8cde150d9\game_script.md`
- **Lore / Backstory:**
  - Real-life friends who play **Lineage 2 Essence** together.
  - Clan **Epidemic** is led by **FrankieDeckard** (Albanian, broken English, heavy smoker, reads Russian forum leaks at 4 AM).
  - Essence gameplay revolves around 24/7 auto-hunting, L-Coins, compounding accessories, and gacha boxes.
  - Frankie constantly misleads the clan into rolling trash classes and wasting resources, which is the source of the bad starter gag.
  - Spanish regional accents (Venezuelan, Canarian, Mexican, Malagueño, Brazilian Portuñol).

---

## 2. Environment & Toolchain Instructions
- **Platform:** Ubuntu on WSL2.
- **Repository Path:** `/mnt/c/Users/ricci/Desktop/code/pokemon-enanced` (accessible from Windows at `C:\Users\ricci\Desktop\code\pokemon-enanced`).
- **Prerequisites (apt):** `build-essential`, `binutils-arm-none-eabi`, `gcc-arm-none-eabi`, `libpng-dev`, `git`.
- **Compiler:** `tools/agbcc/bin/agbcc` (built and installed from `pret/agbcc`).
- **Compilation Command:**
  ```bash
  wsl bash -c "cd /mnt/c/Users/ricci/Desktop/code/pokemon-enanced && make -j\$(nproc)"
  ```
- **Output Binary:** `pokefirered.gba` (32MB GBA ROM).
- **Matching Baseline Check:** Baseline sha1 is `41cb23d8dccc8ebd7c649cd8fbb58eeace6e2fdc  pokefirered.gba`.

---

## 3. Key Technical Conventions & Gotchas

### Character Encoding & Diacritics
- UTF-8 characters (`á`, `é`, `í`, `ó`, `ú`, `ñ`, `¿`, `¡`, `Á`, `É`, `Í`, `Ó`, `Ú`, `Ñ`) are natively parsed by `tools/preproc` via `charmap.txt`.
- Works both in `.c` files via `_("Texto en español...")` and in `.inc` assembly script files via `.string "Texto en español..."`.

### Multichoice Menus
1. Define constant in `include/constants/menu.h`:
   ```c
   #define MULTICHOICE_MY_MENU 65
   ```
2. Declare string pointers in `include/strings.h` and define in `src/strings.c`:
   ```c
   const u8 gText_MyOption[] = _("Option text");
   ```
3. Add `MenuAction` list and `MULTICHOICE(...)` mapping in `src/script_menu.c`:
   ```c
   static const struct MenuAction sMultichoiceList_MyMenu[] = {
       { gText_Option1 },
       { gText_Option2 },
   };
   // in sMultichoiceLists[]:
   [MULTICHOICE_MY_MENU] = MULTICHOICE(sMultichoiceList_MyMenu),
   ```
4. In event scripts, `multichoice left, top, MULTICHOICE_MY_MENU, FALSE`.
5. Cancel button press returns `127` (e.g. `case 127, EventScript_Cancel`).

### Oak Speech / Intro Sequence
- Defined in `src/oak_speech.c` and `data/text/new_game_intro.inc`.
- Intro showcase Pokémon species is controlled by `#define INTRO_SPECIES SPECIES_MAGIKARP` in `src/oak_speech.c`.
- Default player and rival name options are in `data/text/new_game_intro.inc` (`gNameChoice_Red` = "GVR", `gNameChoice_Green` = "KYRALES").

### Starters & Rival Setup
- Starter Pokéballs on table:
  - Ball 0 (x=8, y=4): `SPECIES_MAGIKARP` (Level 5)
  - Ball 1 (x=9, y=4): `SPECIES_METAPOD` (Level 5)
  - Ball 2 (x=10, y=4): `SPECIES_SUNKERN` (Level 5)
- Lab script: `data/maps/PalletTown_ProfessorOaksLab/scripts.inc` and `text.inc`.
- Rival lab battle party: `src/data/trainer_parties.h` (`sParty_RivalOaksLabSquirtle`, `sParty_RivalOaksLabCharmander`, `sParty_RivalOaksLabBulbasaur`).
- Movement collisions: Player stands at y=5 when picking a ball. Rival walks along y=6 to navigate past player without collision.

---

## 4. Work Completed (Phase 1)
- [x] Cloned `pret/pokefirered` and installed `agbcc` toolchain.
- [x] Clean matching baseline build verified.
- [x] Created GitHub repo `riccivr/pokemon-enanced` and pushed baseline.
- [x] Intro sequence updated to FrankieDeckard (Albanian mannerisms, broken English, Magikarp showcase, Lineage 2 lore).
- [x] GVR's house Mom dialogue updated with Venezuelan phrasing and persistent forgetful choices ("No me acuerdo").
- [x] Starter table updated to Magikarp, Metapod, Sunkern (all level 5).
- [x] Kyrales lab cutscenes (tow truck breakdown, complaints about Essence class) and first rival battle configured with the leftover starter.
- [x] Full build verification with 0 errors.

---

## 5. Next Steps (Upcoming Phases)
1. **Phase 2: Route 1 & Viridian City**
   - Nurse Joy replacement: **Nath** (Venezuelan aggressive bully who insults GVR) in Viridian Pokémon Center.
   - Route 22 early rematch with Kyrales.
   - Delivery parcel event for FrankieDeckard.
2. **Phase 3: Pewter Gym**
   - Gym Leader: **Fafa (ntl)** (Rock type, Venezuelan).
3. **Phase 4: Mt. Moon & Nova Rocket**
   - Nova Rocket duo: **wolfitooxd & DvD** (encounter 1).
