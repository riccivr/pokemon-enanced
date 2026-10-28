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

### Trainer Name Buffer Expansion
- In `include/battle.h`, `struct Trainer.trainerName` was expanded from `u8 trainerName[12];` to `u8 trainerName[32];`.
- This accommodates long dual-names like `wolfitooxd & DvD` (and future multi-O variants such as `wolfitooooooooooxd & DvD`).
- The struct is C-managed in ROM and accessed through pointers and EOS termination, so this compiles safely without binary layout breakage.

---

## 4. Work Completed

### Phase 1: Pallet Town Baseline & Prototype
- [x] Cloned `pret/pokefirered` and installed `agbcc` toolchain.
- [x] Clean matching baseline build verified.
- [x] Created GitHub repo `riccivr/pokemon-enanced` and pushed baseline.
- [x] Intro sequence updated to FrankieDeckard (Albanian mannerisms, broken English, Magikarp showcase, Lineage 2 lore).
- [x] GVR's house Mom dialogue updated with Venezuelan phrasing and persistent forgetful choices ("No me acuerdo").
- [x] Starter table updated to Magikarp, Metapod, Sunkern (all level 5).
- [x] Kyrales lab cutscenes (tow truck breakdown, complaints about Essence class) and first rival battle configured with the leftover starter.
- [x] Full build verification with 0 errors.

### Phase 2: Route 1 through Mt. Moon
- [x] **Viridian Mart & Frankie's Parcel Delivery:**
  - Viridian Mart clerk hands over Frankie's carton of Marlboro Reds and USB drive of 4 AM Russian Lineage 2 forum dumps.
  - Delivery back to Frankie in Pallet Lab updated with broken English rejoicings, giving the Epidemic Clan Strategy Guide (Pokédex) and Poké Balls.
  - Kyrales dialogue updated (watching YouTube tow truck repair tutorials, leaving to get tow truck parts in Viridian).
- [x] **Nurse Joy (Nath's Bullying Counter Script):**
  - Updated `data/scripts/pkmn_center_nurse.inc` and `data/event_scripts.s`.
  - Nath roasts GVR on entry ("das pena ajena", asks if his team wiped again).
  - Healing branch insults him and tells him not to trip on his way out.
  - Cancel branch screams at him for wasting her time and threatens to throw a potion at his forehead.
- [x] **Pewter Gym Leader (Fafa ntl):**
  - Replaced Brock with `FAFA` in `src/data/trainers.h` and `data/maps/PewterCity_Gym/text.inc`.
  - Dialogue warns GVR that he isn't in a safe auto-hunt zone, to sweat for the Boulder Badge, and tells him to go to Cerulean to learn how to farm properly.
- [x] **Mt. Moon B2F (Encounter 1 with wolfitooxd & DvD):**
  - Expanded `trainerName[32]` in `include/battle.h`.
  - Configured `TRAINER_TEAM_ROCKET_GRUNT` as `wolfitooxd & DvD`.
  - Party configured with Level 70 Magikarp (Splash) and Level 12 Zubat.
  - Pre-battle, defeat, and post-battle dialogue implements the Baium Doll lvl 3, +7 talisman, and Olympiad meta rants.
- [x] **CI & Verification:**
  - `make -j$(nproc)` compiles with 0 errors.
  - GitHub Actions workflow successfully builds and uploads `pokemon-enanced.gba` artifact for `phase2-viridian-pewter-mtmoon`.

---

## 5. Next Steps (Upcoming Phases)
1. **Phase 3: Cerulean City, Nugget Bridge & Route 24/25**
   - Cerulean Gym Leader (Water type).
   - Nugget Bridge trainers and Rocket recruitment pitch.
   - Bill's Sea Cottage.
2. **Phase 4: Vermilion City & S.S. Anne**
   - Kyrales broken tow truck arrival scene outside Vermilion Gym.
   - Lt. Surge replacement: **Omar (Thunderay)** (all Shiny team, extreme L-Coin gacha luck).
   - S.S. Anne rival battle and Cut HM.
3. **Phase 5: Celadon City & Rocket Game Corner**
   - Erika replacement: **Mauro** (AFK auto-hunt gym leader, gives badge on second prompt).
   - Rocket Game Corner B4F: **wolfitooooxd & DvD** (Encounter 2, with 4 O's).
