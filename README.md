# Bookworm

**Read books, raise insects.** Bookworm is a reading tracker built as a virtual pet. Every book you start hatches an insect larva. Each page you read feeds it, and it grows through its life stages. When you finish the book, it emerges as an adult and joins your Species Collection, along with field notes about the real insect.

It's meant to be educational and friendly for beginners in entomology. Every species card is fact-checked, links to free guides from conservation organisations, and lists its sources.

> **Status:** single-file HTML prototype (October 2026). The plan is to turn it into an Android app. Please read [Before the Android build](#before-the-android-build) first.

---

## Contents

- [Playing it](#playing-it)
- [How the game works](#how-the-game-works)
- [The species](#the-species)
- [Countries: United Kingdom and Ireland](#countries-united-kingdom-and-ireland)
- [EPUB Library and reader](#epub-library-and-reader)
- [Species cards](#species-cards)
- [Testing notes, October 2026](#testing-notes-october-2026)
- [Fact-checking and sources](#fact-checking-and-sources)
- [Language and units](#language-and-units)
- [Book search](#book-search)
- [Project files](#project-files)
- [How the code is organised](#how-the-code-is-organised)
- [Testing](#testing)
- [Before the Android build](#before-the-android-build)
- [Design decisions log](#design-decisions-log)

---

## Playing it

- **Online:** open `index.html` from GitHub Pages, or the claude.ai artifact version.
- **Locally:** open `index.html` in any modern browser. No install or build step is needed.
- **Saves:** progress is stored in the browser's `localStorage`. Clearing site data wipes it, and each browser or device has its own save.

## Saving, testers and moving progress

Progress lives in the browser's own storage (`localStorage`) for the site it's opened from, e.g. `sauvryn.github.io`.

**A tester keeps their progress as long as they use the same browser on the same device and open the same link.** Updating `index.html` on GitHub doesn't wipe saves: the new version reads the same storage keys.

Progress is lost or hidden in these cases:
- **A different device or browser:** each has its own separate save, and nothing syncs between them.
- **Private or incognito windows:** the save is thrown away when the window closes.
- **Clearing browsing data:** clearing cookies and site data deletes the save.
- **iPhone and iPad (Safari):** storage can be deleted after about a week without a visit. Adding the page to the home screen ("Share" → "Add to Home Screen") usually prevents this.
- **Other projects on the same GitHub Pages site:** they share the same storage area. All Bookworm keys start with `bookworm`, so they won't clash unless another project uses those names.

### Export / Import Save (Settings)
- **Export** writes every `bookworm…` storage key to a dated file, e.g. `bookworm-save-2026-10-03.json`. That covers every country's save, the country choice, the Merge Collections setting, meta achievements and titles.
- **Import** reads a save file, shows when it was exported and how many completed books it holds, warns that it **replaces all Bookworm progress in this browser**, then loads it and reloads the page. Storage that doesn't belong to Bookworm is left alone.
- **Copy and paste instead:** a fallback for browsers that can't download or pick files. "Copy my save" puts the save text on the clipboard (or in a box to copy by hand), and "Import pasted save" loads pasted text.
- **Uses:** moving a tester to a new device, backing up before a big test, or loading a tester's save to reproduce a bug.
- **The claude.ai preview:** downloading doesn't work there, so use copy and paste. Downloads work on GitHub Pages.
- **Save file format:** `{app:"bookworm-save", format:1, version, exported, data:{key: value}}`. Import rejects anything else, or any key that doesn't start with `bookworm`. For the Android build, keep this format, or write a converter, so testers' saves can move into the app.

## How the game works

### Books and larvae
- **One larva per book.** Starting a book hatches an egg. The larva's species is chosen at random, as described in [Choosing a species](#choosing-a-species).
- **Logging progress.** You log reading as a **page number** or a **percentage**, and you can switch between the two at any time. The app converts between them, rounding up, and the larva never loses a stage when you switch.
- **Growth.** The larva grows through 8 stages, each with its own congratulations screen and a fact. It then pupates (a chrysalis for butterflies, a pupa for everything else) at a **species-specific point** in the book, and the adult emerges at **100%**.
  - **Pupation point:** worked out from the species' real active development time, as larva days ÷ (larva + pupa days), with winter dormancy left out. The Peacock pupates at 63%, most species fall between 50% and 90%, and multi-year beetle larvae like the Stag Beetle pupate at about 95%. A few parasitic insects with short larval lives pupate at around a third of the way through.
  - **Life cycle row:** each species card shows the active days as a larva and as a pupa, and flags figures estimated from close relatives.
  - **Winter field note:** each species has a field note saying which stage spends the winter.
  - **Durations:** 100 species have species-level figures, 37 genus-level and 87 family-level estimates. They're in `DUR`.
- **Pupae don't feed.** Once it pupates:
  - the Feed button reads **Morph** (for metamorphosis), and logging reading shows "Morphing!" instead of "Munch!". The mood label stays "Pupating", the scientific word;
  - the hunger status shows **Pupating**, and a pupa never gets hungry;
  - queen or worker is decided by feeding during the larval stage only.
- **Correcting progress.** If you enter the wrong page, the page-number editor lets you fix it.
- **Logging reading.** The Log Reading box sits right under the larva, so you can watch it munch when you press **Feed**. Type the page you're on, or your percentage, and press Feed.
- **Field notes while raising.** Each larva's page has a **See Field Notes** button under its hunger status. It opens a pop-up with the species' field notes, gardening tip and links as soon as the egg hatches. In the Species Collection, the field notes stay locked until you've raised the adult.
- **Reading screen lists.** Below Your books there's a **Want To Read** list and a **Completed Books** list. Each shows the three most recent titles, and tapping one opens the whole list.
- **Want To Read.** The Add a book page has an **Add to Want To Read list** button under "Next: choose how to track", and you can use it even when you're already reading 5 books. Each entry in the full list has **Start Reading**, which opens the "How do you want to track this book?" pop-up, and **Remove**.
- **Re-reading.** You can add the same book again, because people re-read books. If a book matches one you're reading, have finished, or have on your list, the app first asks "You have added this book before. Are you reading it again?", with **Yes** or **No**. A book matches if the ISBNs match, or the titles match and the authors match (or one is blank).
- **Nothing dies.** If you stop reading, the larva just gets "extremely hungry".

### Rarity tiers (by page count)

| Book length | Tier |
|---|---|
| under 450 pages | Common |
| 450–799 pages | Uncommon |
| 800+ pages | Rare |

### Shelf limits
- **Books:** you can read up to **5** books at once.
- **Frozen:** you can freeze up to **5** books (paused reading). Frozen larvae don't grow.
- **Waiting:** up to **5** larvae can wait without a book, and they don't take up a reading slot. Tap one to see what kind of insect it is (for example "common wasp"), or to release it.
- **Abandoning a book:** you choose to either **release** the larva or **transfer** it to another book of a similar tier. A transfer gives you a random eligible larva and resets its growth.

### Choosing a species
1. **Unlocked categories only.** Every egg starts out as a butterfly. The other categories unlock as you raise butterflies (repeats count):

   | Category | Unlocks after raising |
   |---|---|
   | Butterflies | from the start |
   | Moths | 2 butterflies |
   | Bees & wasps | 4 butterflies |
   | Beetles | 6 butterflies |
   | Flies | 8 butterflies |
   | Ants | 10 butterflies |
   | Other insects | 12 butterflies |

   Unlocks are shown on the Achievements screen, get a banner on the "emerged" screen, and appear as a note on locked collection tabs.
2. **All uncollected species of the book's tier, ignoring category.** Bigger categories come up more often and small ones like ants fill slowly. (The first prototype picked a category first with equal odds, which gave out ants far too quickly.)
3. **Season.** Species that are **in season this month** are weighted **3:1**, and they get an "In season now!" chip.
4. **Exclusions.** It skips any species currently being raised, frozen or waiting.
5. **Fallback.** Only when nothing new is left in the tier can a repeat hatch.

"This month" comes from the device clock. The **Prototype tools** panel can override it for testing.

### Male and female forms
32 species have adult males and females that a beginner could tell apart from a photo, for example the Orange-tip, Common Blue, Vapourer, Stag Beetle and Scorpionfly. The rules for choosing them are the same as the look-alike rule: obvious differences only, so no sex brands, eye spacing or hand-lens features.

- **Hatching.** These species hatch as a **male** or a **female**. The larva's page shows ♂ Male or ♀ Female under its length.
- **Collecting.** Raising either form counts the species as collected, so its field notes unlock and it counts towards achievements.
- **The other form.** The collection shows which forms you have. The missing form still counts as "uncollected" when an egg is picked, and a pick gives you the missing form.
- **Species card.** It describes every form. Once forms are raised, it shows art for each.
- **One at a time.** Only one larva of a species can be raised at once, even when another form is still missing.

### Queens and workers
19 social species have **queen** and **worker** forms: all 5 ants, the Honey Bee, all 11 bumblebees, the Common Wasp and the Hornet. The Red-tailed Bumblebee also keeps its distinctive male (yellow face and bands), so it has three forms.

- **No other castes.** Ordinary males of the other social species aren't included, because they're short-lived and look too much like the females. There are no soldiers either, because no British or Irish ant has a true soldier caste.
- **Hatching.** A larva of these species hatches as a female, apart from the Red-tailed Bumblebee male.
- **Queen or worker? It depends on feeding, as in real colonies.**
  - She becomes a **queen** if she never gets **Hungry** while her book is read.
  - She becomes a **worker** if she does.
  - The larva's page tells you which way she's heading.
  - The "emerged" screen explains the real science: royal jelly for honey bees, extra food late in the season for bumblebees, special well-fed queen cells for wasps, and nutrition (plus other factors in some species) for ants.
- **Spring queens.** Bumblebees, the Common Wasp and the Hornet have a field note explaining that the big ones seen in early spring are queens fresh out of hibernation.
- **Old saves.** Larvae and adults in older saves were given a form. Previously raised social species became workers, and larvae became females whose caste is still to be decided.

### I've Seen It!
On any species card you can mark which life stages you've seen in the wild: egg, larva, pupa or adult. Seen species get a marker in the collection. This works even for species you haven't raised yet.

Species with male and female, or queen and worker, forms get a separate line for each adult form, e.g. "Adult male butterfly" and "Adult female butterfly", or "Queen ant" and "Worker ant". That way you can record seeing each one. A plain "adult" tick saved before this change still shows, as "Adult (form not recorded)". Ticking a stage shows an editable **Date seen**, which defaults to today and can't be in the future. The species card shows the dates, e.g. "You've seen: adult (3 Oct 2026)". Dates are stored in `state.seenDates`, and Reset App clears them. Unticking a stage first asks "Are you sure you want to remove this sighting?" (Keep it / Remove), in case of a stray tap.

### Habitats
- **The habitats:** 8 broad habitats, based on the Wildlife Trusts' habitat groups:
  - Woodland;
  - Grassland & Meadows;
  - Heathland & Moorland;
  - Wetlands & Bogs;
  - Rivers, Ponds & Lakes;
  - Coast;
  - Farmland & Hedgerows;
  - Towns, Gardens & Homes.
- **Which species live where:** each species lives in 1–3 of them, main habitat first. They're in `HAB`, sourced mostly from Butterfly Conservation, BWARS, the Bumblebee Conservation Trust, NatureSpot and Wikipedia.
- **Unlocking:** each adult you raise unlocks **one** habitat at most: the first habitat on its species card that's still locked. If that one is already open, it unlocks the second, and so on. So habitats open one at a time, but a habitat needs only one raised species that lives there. Each unlock gets a banner on the "emerged" screen.
  - **Older saves:** these were recalculated once with this rule, replaying finished books oldest first.
  - **Merging and resetting:** merged collections replay the same way. Unmerging re-locks habitats that only opened through merging, and Reset App clears them all.
- **Where they are:** the Collection screen has a **Species / Habitats** switch. Each unlocked habitat opens a simple placeholder screen: its name and a list of the species that live there, listed in-season species first, then out-of-season ones, alphabetically within each group. Raised species are green, bold and ticked. Species not yet raised are orange, to make the difference obvious. Species in season this month get the leaf mark, so you can see what should be active there now. Tapping a species name opens its species card, and closing the card returns to the habitat. Species cards also list their habitats as links: tapping one opens that habitat (or, if it's still locked, says how to unlock it).

### Score and titles
- **Points:** every finished book earns points for its species' rarity in the country it was raised in: **100** Common, **200** Uncommon, **300** Rare. Repeats count. Completed Books and species cards show each book's points in that rarity's colour.
- **Score:** the score covers every country (merging doesn't change it), and it's shown under the app title, with the current title beneath it. Tapping it opens the Titles list on the Achievements screen.
- **Titles:** these are earned at score thresholds, and each new one gets a banner on the "emerged" screen.

  | Points | Title |
  |---|---|
  | 0 | Egg Explorer |
  | 500 | Hungry Hatchling |
  | 1,500 | Caterpillar Chapter-Chaser |
  | 3,000 | Larva Librarian |
  | 5,000 | Paperback Pupa |
  | 8,000 | Bookish Butterfly |
  | 12,000 | Moth of the Margins |
  | 17,000 | Bumblebee Bibliophile |
  | 23,000 | Hawk-moth Historian |
  | 30,000 | Lepidoptera Laureate |
  | 40,000 | Entomology Encyclopaedist |
  | 55,000 | Metamorphosis Maestro |
  | 75,000 | Monarch of the Manuscripts |

  For scale, raising every form of every species is worth about 43,800 points in the United Kingdom and 34,800 in Ireland, so the top title needs both countries, or a lot of re-reading.
- **Storage:** titles are stored in `bookworm-titles`, and older saves were scored retroactively.

### Achievements
- **Medals:** every achievement is earned in stages: **Copper, Bronze, Silver, Gold, then Platinum**. The stages are spread evenly over the total: `medalSteps(total)` gives thresholds `ceil(k × total / 5)`.
  - Platinum always means all of it.
  - With fewer than 5 to collect, the lowest medals are skipped: 2 species give Gold at 1 and Platinum at 2; 3 give Silver, Gold, Platinum.
  - Each card shows the current medal in its colour, a row of pips (one per stage), the date of the latest medal, and what the next one needs.
  - Newly earned medals appear on the "emerged" screen, and as a toast when a sighting earns one.
- **Collection Achievements** (each country's own save, `state.achievements`):
  - one "collect them all" per category, with country names such as "Butterfly House";
  - **By rarity:** every Common, Uncommon or Rare species in each category, plus in the whole country. These are grouped in three fold-out sections;
  - **Sightings:**
    - "Keen Spotter": mark 50 species as seen at any life stage.
    - "Life Cycle Watcher": see the larva, the pupa and at least one adult form of 5 species.
- **Meta achievements** (shared by all countries, `bookworm-meta`):
  - each country's whole collection;
  - "Common Ground", "Uncommonly Good" and "Rarities of the World": every Common, Uncommon or Rare species in every country, with each country's own rarity.
- **How medals are saved:** saved records are `{m: medal index, dates: {medal: date}}`. Older saves that stored just a completion date are read as Platinum. `checkAchievements()` keeps saved medals in line with progress. Medals that no longer count, for example after unmerging, are taken away quietly, the next time that country is opened.

## The species

### Rules for inclusion
1. **Complete metamorphosis only.** Every species goes through egg, larva, pupa and adult. Insects such as dragonflies, grasshoppers and true bugs are left out.
2. **Novice-distinguishable only.** A species gets its own entry only if a beginner could tell it apart by eye or from a decent photo. Near-identical look-alikes are explained in the main species' field notes instead:

| Collectible | Look-alike folded in (old save ID → new) |
|---|---|
| Small Skipper | Essex Skipper (`essex-skipper` → `small-skipper`) |
| Wood White | Cryptic Wood White (`cryptic-wood-white` → `wood-white`) |
| Wood Ant | Southern, Hairy and Scottish Wood Ants (`southern-wood-ant`, `hairy-wood-ant` → `wood-ant`) |
| Crane Fly | renamed from "Daddy Longlegs" (`daddy-longlegs` → `crane-fly`), so it isn't mistaken for a spider |

Old saves are migrated automatically through the `MERGED` alias map.

### Counts

| Category | United Kingdom | Ireland |
|---|---|---|
| Butterflies | 53 | 34 |
| Moths | 54 | 46 |
| Bees, wasps & sawflies | 31 | 27 |
| Beetles | 48 | 36 |
| Ants | 5 | 4 |
| Flies | 23 | 19 |
| Other insects (lacewings, caddisflies, scorpionflies, alderflies, fleas and so on) | 9 | 6 |
| **Total** | **223** | **172** |

### Where the species came from
- **Starting lists:** the Wildlife Trusts' *Wildlife Explorer* lists for each category.
- **Added for Northern Ireland and Ireland:** extra species, for example the Narrow-bordered Bee Hawk-moth, plus Ireland-only species such as the **Burren Green**.
- **Added after a gap check:** sawflies, lacewings, more hawk-moths and tigers, ladybirds, hoverflies and bees.
- **Irish checks:** every British species was checked against Irish records, mainly National Biodiversity Data Centre maps, the Irish Red Lists, MothsIreland and the All-Ireland Pollinator Plan. Species with no Irish records, or only one-off strays, are left out of the Irish collection.
- **The Monarch** is the app's original species. It stays in both collections as a rare vagrant.

## Countries: United Kingdom, Ireland, the United States and Canada

- **Switching country.** Tap the flag in the top-right corner to change country. The page reloads with that country's data.
- **What's separate.** Each country has its own species list, rarity tiers, status lines, field notes, save and achievements. The saves are `bookworm-proto-v1`, `bookworm-proto-v1-ie` and `bookworm-proto-v1-us`, and the chosen country is stored in `bookworm-country`.
- **United States (first pass complete).** The collection has 243 species, across all seven categories.
  - **Butterflies (67):** 62 American species plus 5 shared with the UK. The shared ones are the Monarch, Painted Lady and Red Admiral, plus the Small White and Small Copper, which go by their American names, Cabbage White and American Copper.
  - **Moths (59):** 51 American species plus 8 shared with the UK, using American rarity and notes. The shared moths are the Garden Tiger, Large Yellow Underwing, Peppered Moth, Cinnabar (released in the Pacific Northwest to control ragwort), Ruby Tiger, Herald, Winter Moth and Box-tree Moth (the last two are invasive). The American moths range from silk moths (Luna, Cecropia, Polyphemus, Regal) and hawk moths to the woolly bear, Yucca Moth, Black Witch and pests such as the Spongy Moth and Indian Meal Moth.
  - **Other insects (15):** 8 American species (Eastern Dobsonfly, whose larva is the hellgrammite; Antlion, the doodlebug; Owlfly; Snakefly; Wasp Mantidfly; Fishfly; Hanging Scorpionfly; Earwigfly) plus 7 shared with the UK (Common Green Lacewing, Brown Lacewing, Cat Flea, Caddisfly, Scorpionfly, Alder Fly, Snow Flea). The Dobsonfly, Snakefly and Earwigfly have male and female forms. The achievement is "Backyard Curiosities", and the category unlocks after 12 butterflies. Their "Want To Learn More?" links go to a BugGuide search (Iowa State University), because Butterflies and Moths of North America only covers butterflies and moths.
  - **Ants (10):** 9 American species plus the UK's Red Ant, which is called the European Fire Ant in the US, where it's an invasive pest. The American ones are the Black Carpenter Ant, Red Imported Fire Ant, Odorous House Ant, Pavement Ant, Argentine Ant, Red Harvester Ant, Honeypot Ant, Allegheny Mound Ant and Texas Leafcutter Ant.
    - **Queens and workers:** all have queen and worker forms, decided by feeding as elsewhere. Their descriptions are in `US_ANT_CASTE`.
    - **Achievement and unlock:** the achievement is "Anthill Americana", and the category unlocks after 10 butterflies.
    - **Life cycle:** each uses a rough 21-day larva and 21-day pupa estimate.
    - **Possible addition:** leafcutter ants have true big-headed soldiers (majors), so a "soldier" form could be added for them later.
  - **Flies (22):** 18 American species plus 4 shared with the UK: Drone-fly, Crane Fly, Narcissus Bulb Fly and Dark-edged Bee-fly.
    - **The American species:** House Fly, Black Soldier Fly, American Hover Fly, Transverse Flower Fly, Virginia Flower Fly, Red-footed Cannibalfly, Deer Fly, Black Horse Fly, Greenhead, Common Green Bottle Fly, Common Fruit Fly, Asian Tiger Mosquito, Feather-legged Fly, Golden-backed Snipe Fly, Phantom Crane Fly, Long-legged Fly, Rabbit Bot Fly and Mydas Fly.
    - **Forms:** there are no male and female forms, because the main differences in flies are eye spacing or antennae, which the rules exclude.
    - **Achievement and unlock:** the achievement is "Fly-Over Country", and the category unlocks after 8 butterflies.
  - **Beetles (41):** 36 American species plus 5 shared with the UK.
    - **Shared with the UK:** the 7-spot, Harlequin, 2-spot and 14-spot ladybirds appear under their American "lady beetle" names; the Harlequin is called the Asian Lady Beetle. The Devil's Coach Horse is the fifth.
    - **The American species:** they include the Eastern Hercules Beetle, Giant and Reddish-brown Stag Beetles, the Japanese Beetle, Green June Beetle, Rainbow Scarab, Eastern Eyed Click Beetle, Big Dipper and Synchronous Fireflies, Six-spotted Tiger Beetle, Colorado Potato Beetle, Red Milkweed Beetle, Golden Tortoise Beetle, Emerald Ash Borer, Asian Longhorned Beetle, American Burying Beetle, Bess Beetle, Bombardier Beetle, Convergent Lady Beetle, Diabolical Ironclad Beetle and Boll Weevil.
    - **Male and female forms:** five have them (Hercules Beetle, both stag beetles, Rainbow Scarab and Ten-lined June Beetle).
    - **Achievement and unlock:** the achievement is "Lightning Bug Legion", and the category unlocks after 6 butterflies.
  - **Bees & wasps (29):** 26 American species plus 3 shared with the UK.
    - **Shared with the UK:** the Honey Bee, the European Hornet (the UK's Hornet) and the European Wool Carder Bee.
    - **The American species:** they include 5 bumblebees, among them the endangered Rusty Patched Bumblebee, plus the Eastern Carpenter Bee, Blue Orchard Mason Bee, Squash Bee and Bicolored Striped Sweat Bee. Wasps include the Eastern Yellowjacket, Bald-faced Hornet, two paper wasps, the Great Golden Digger Wasp, Eastern Cicada Killer, Tarantula Hawk, mud daubers, the Eastern Velvet Ant (a wingless wasp), the Giant Ichneumon Wasp and the Pigeon Tremex.
    - **Queens and workers:** the 5 bumblebees, the yellowjacket, the Bald-faced Hornet and both paper wasps have them, decided by feeding. Paper wasps have their own "Why a queen?" text in `CASTE.why.paperwasp`, because they don't build special queen cells.
    - **Male and female forms:** five species have them: the Carpenter Bee, the sweat bee, the Velvet Ant, the Giant Ichneumon and the Pigeon Tremex.
    - **Achievement and unlock:** the achievement is "Stars, Stripes & Stingers", and the category unlocks after 4 butterflies.
  - **Moth forms and unlocks:** six moths have male and female forms (Promethea, Io, Salt Marsh, Spongy, White-marked Tussock and Bagworm). In the US, moths unlock after 2 butterflies, as elsewhere. Their achievement is "Porch-Light Parade".
  - **How it was written:** the American species notes, rarity, sizes, seasons, life cycles, habitats, gardening tips and male/female forms were first **written from general knowledge**. They're being fact-checked one category at a time, smallest first, against US sources (see [Fact-checking American species](#fact-checking-american-species)). Cards that haven't been checked yet say so.
  - **Fact-check progress:** every American category is checked ✅ (Ants, Other insects, Flies, Bees & Wasps, Beetles, Moths and Butterflies, October 2026). Larva and pupa durations and habitats (`US_DUR`, `US_HAB`) haven't been checked yet; they could get their own pass.
  - **Language:** British or American English can be chosen separately from the country. See [Language and units](#language-and-units).
  - **Look-alikes:** these are folded in the same way as for the UK and Ireland, e.g. Canadian into Eastern Tiger Swallowtail, Eastern Comma into Question Mark, Northern into Pearl Crescent, Five-spotted Hawk Moth into Carolina Sphinx, Snowberry into Hummingbird Clearwing, and Forest into Eastern Tent Caterpillar Moth.
  - **What it has:** category achievements ("Star-Spangled Wings" for butterflies, "Porch-Light Parade" for moths) and a meta achievement, "Collect All American Species".
  - **Learn More links:** Butterflies and Moths of North America (each species page was checked), the North American Butterfly Association and the Xerces Society.
  - **Habitats:** the same eight habitats have American names and descriptions (`HABITATS_US`): Woodland & Forest; Prairies, Meadows & Deserts; Mountains & Shrublands; Wetlands & Swamps; Rivers, Ponds & Lakes; Coasts & Beaches; Farms & Orchards; and Backyards, Towns & Homes.
  - **Inches:** every size shows in inches first, with metric in brackets, e.g. "3.1–5.5 in (79–140 mm)". That covers wingspans, body lengths, sizes in facts and form descriptions, the larva's length chip and the growth screens. The UK and Ireland show metric first with imperial in brackets. See [Language and units](#language-and-units).
  - **Still to do:**
    - verify the BugGuide search links;
    - check American larva and pupa durations and habitats.

    The Fall Webworm and Virginian Tiger Moth pages on Butterflies and Moths of North America were confirmed in the Moths check.
- **Coverage.** The **United Kingdom collection includes Northern Ireland**. Both countries get the complete experience.
- **Wording, chosen to respect cultural identity:**
  - The country picker reads **"United Kingdom"** and **"Ireland"**.
  - Species are described as **"British"** or **"Irish"**, as in "223 British species".
  - "UK" is fine inside field notes for brevity.
- **Irish mode details:**
  - Rarity comes from `IE_RARITY`.
  - Irish status lines and an optional Irish fact come from `IE_NOTES`. The default status lines are in `IE_STATUS`.
  - Facts that mention British places are hidden automatically by `PLACE_RE`.


### Canada (added October 2026)
- **How it was written:** from general knowledge and **not yet fact-checked**. The plan is to check it after the American checks are finished. Cards say what has and hasn't been checked.
- **195 species:**

  | Category | Species |
  |---|---|
  | Butterflies | 44 |
  | Moths | 48 |
  | Bees & Wasps | 27 |
  | Beetles | 35 |
  | Flies | 21 |
  | Ants | 6 |
  | Other insects | 14 |

- **Mostly shared:** nearly all are shared with the American list, and some with the British one. They keep their checked field notes, minus facts about American places (`CA_PLACE_RE`, all states except where a name like "Virginia creeper" is a plant), Britain or Ireland. Each one gets a Canadian rarity, status and note from `CA_NOTES`: `id: [C/U/R, status, note, {name, sci, food, season}]`.
- **Shared British species:** these take their American names and the checked American rewrites of their notes, as in the US, unless Canada has its own name.
  - **Canadian Tiger Swallowtail** (*Papilio canadensis*) is the same collectible as the Eastern Tiger Swallowtail, which reaches only southern Ontario.
  - **White Admiral** is the banded form of *Limenitis arthemis*, the same collectible as the Red-spotted Purple. It's a different species from Britain's White Admiral.
  - **Old World Swallowtail** is the British Swallowtail, found in the North and West, with its own food plants.
  - **European Skipper** (*Thymelicus lineola*) uses the British Small Skipper collectible. It was introduced to Ontario around 1910.
  - **Elephant Hawk-moth** is introduced in British Columbia.
- **Canada-only species (`CA_ONLY`, 8):**
  - Macoun's Arctic;
  - Spruce Budworm;
  - Arctic Woolly Bear Moth (up to seven winters as a caterpillar);
  - Mountain Pine Beetle;
  - Whitespotted Sawyer (male and female forms);
  - Yellow-banded Bumblebee (queen and worker);
  - Western Thatching Ant (queen and worker);
  - Black Fly.
- **Language: Canadian English** (`caSpell()`, `CA_WORDS`):
  - British spellings ("colour", "centre", "grey", "metre", "travelled") but -ize endings ("recognize", "analyze");
  - American spelling for a few words (aluminum, fetus, cozy, skeptic);
  - North American words: fall, sidewalk, garbage, lady beetle, hover fly, and two-word true flies;
  - gardening tips keep "garden".
- **Language prompt:** switching to or from Canada asks about Canadian English.
- **Units:** metric first, imperial in brackets, as in the UK and Ireland.
- **Dates:** `en-CA` style.
- **Habitats** (`HABITATS_CA`): Forests & Woodlands, Prairies & Meadows, Mountains & Tundra, Wetlands & Bogs, Lakes, Rivers & Ponds, Coasts & Beaches, Farms & Orchards, and Backyards, Towns & Cities.
- **Achievements** (with medals):
  - From Sea to Sea (butterflies);
  - Cottage Porch Light (moths);
  - The Maple Hive (bees and wasps);
  - Boreal Beetles;
  - Anthills of the Shield;
  - Black Fly Season;
  - Curiosities of the True North (other insects);
  - the meta achievement "Collect All Canadian Species".
- **Learn More links:** as in the US: Butterflies and Moths of North America, or a BugGuide search, plus NABA and Xerces.
- **Flag:** red-white-red with a simplified maple leaf.
- **To fact-check later:** every Canadian rarity, status and note, the Canada-only species, and Canadian seasons, which are often shorter than American ones.

## Settings

Tap the gear under the flag to open Settings.

- **About:**
  - what the app does and its learning objectives, worded for any country ("the selected country's common butterflies…"), with the list of countries built from `COUNTRIES`;
  - **Privacy:** no information is collected, everything stays in this browser on this device, and no account is or ever will be needed for the GitHub-hosted version;
  - **Donate:** Bookworm is free and made for public education; a Ko-fi link (https://ko-fi.com/sauvryn) for donations towards personal development costs;
  - the version number (`APP_VERSION`, currently 0.10.0) and the GitHub Pages link.
- **Merge Collections (Easy Mode):** shares everything you've collected, past and future, across every country.
  - **How it works:** each raised adult records the country it was collected in (`country`). Nothing is copied. While merged, the app reads every country's saves together (`coll()`). So species raised in the United Kingdom that also live in Ireland count as collected in Ireland, which leaves only the rest, including Ireland-only species. This works for any country added later.
  - **What changes on screen:** the collection shows an "Easy Mode" chip. Species cards and Completed Books show where each one was collected. Category unlocks also count butterflies from every country.
  - **Unmerge Collections:** the menu item changes to this once merged. Unmerging goes back to each country's own collection. It removes any category achievements, meta achievements and category unlocks that were only complete because of merging. Nothing else depends on them, so nothing breaks.
- **Language:** British English or American English, independent of country. See [Language and units](#language-and-units). Reset App doesn't change it.
- **Epub Reader:** explains the built-in reader and has the **automatic tracking** switch (on by default, stored in `bookworm-epub-auto`). See [EPUB Library and reader](#epub-library-and-reader).
- **Export / Import Save:** **Export** saves every `bookworm…` storage key to a dated `.json` file: every country's progress, the country choice, merge setting, meta achievements and titles. **Import** loads one, after a warning that it replaces everything in this browser. A copy-and-paste option covers browsers that can't download or pick files. Use it to move a tester to a new device, or to load a tester's save and reproduce a bug.
- **Reset App:** asks twice ("Are you sure? This cannot be undone."). In every country it then deletes:
  - collections and completed books;
  - achievements, meta achievements, titles, the score and category unlocks;
  - waiting larvae;
  - I've Seen It! ticks.

  It also switches Merge Collections off. A notice explains that the Want To Read list, frozen larvae and current books are kept and must be removed by hand.

## Testing notes, October 2026

Changes made from a batch of testing notes (21 items).

### Reading screen and books
- **Fed today:** a small orange heart appears before the mood chip on any book whose reading was logged today (`b.loggedAt`, `fedToday()`).
- **Want To Read:**
  - The buttons sit in a column on the right, so titles have room.
  - ▲ and ▼ buttons reorder the list.
- **Completed Books:**
  - Each book shows the dates it was read, start to finish ("1 Sept 2026 - 3 Sept 2026"). Time spent frozen isn't taken off. Books finished before this change only show the finish date.
  - A star marks favourites. Favourites are kept for every country in `bookworm-favs`.
  - Tapping a book in the full list opens the species card for the insect it raised. Closing it goes back to the list.
- **Dates:** these follow the selected country's standard (`DATE_LOCALE`): "8 Oct 2026" in the UK and Ireland, "Oct 8, 2026" in the US.

### Taking books to another country
- **When it asks:** switching country while reading asks first what to do with those larvae and pupae. This comes before the language question.
- **The four choices:**
  - **Freeze them:** only if there's room among the 5 frozen books.
  - **Take them with me:** only if the other country has room among its 5 current books.
  - **Leave them reading here:** they can still get hungry while you're away.
  - **Cancel.**
- **A larva that's taken along:**
  - It keeps its **home country** (`b.origin`) and a snapshot of its species (`b.sp`): the text before conversion, plus its Learn More links and sources as shown at home. Its name and field notes are converted to the current language and units (`spOf(b)`).
  - A small flag of its home country sits on the top left of its artwork, on the shelf, on its reading page and on the celebration screens.
  - When it emerges, the adult and the completed book are credited to the **home** country's collection (`creditHome()`), even if this country has the same species. That may also open a habitat there.
  - Released while away, it's "safely sent back to its home country to be responsibly released into the wild in its native environment". It can't be moved to the waiting list while away.
  - Taken back home, it becomes an ordinary book again and the flag disappears.

### Collection
- **Collected tiles:** these show the rarity as a lettered badge on the top left (a light letter in a darker circle, in the rarity colours) and how many were raised at the top centre, level with the in-season leaf.
- **Tile artwork:** the tile shows one adult form (`preferredForm()`). The art is placeholder for now, so it doesn't show yet.
  - Male/female species: the more eye-catching form (male unless `PREFERRED_FORM` says otherwise) if raised, otherwise the other.
  - Social insects: the worker if raised, otherwise the most recently raised form.
- **Species cards:**
  - Adult / Chrysalis (or Cocoon, Puparium, Pupa) / Caterpillar (or Larva) buttons switch the artwork. Adults still show every form at once.
  - Additional Sources no longer repeats links already under Want To Learn More?, and the note says "checked against all of the above sources".
- **Search:**
  - **What it searches:** the box above the categories searches common and scientific names (or parts) in every country. This country's species come first. Others show their country's flag and name.
  - **Tapping a result:** this country's results open the species card. Others explain which collection they're in.
  - **The index:** other countries come from `SEARCH_INDEX`, generated by **`node build_index.js`**, which loads the page once per country with `?dump-index`. Run it after changing species data. `index_test.js` fails if it's out of date.

### Achievements
- "Achievements" is now **Collection Achievements**. Medal stages and the new rarity, sightings and meta achievements are described in [Achievements](#achievements).

### Wording
- Text that assumed only the UK and Ireland now works for any country, ready for Canada. For example: About, the Language setting note.

### Splash page
- **What it shows:** "Bookworm" and the tagline "Read books. Raise insects." over a scatter of placeholder insects.
- **When:** for 3 seconds, only when the app is opened in a new window or tab (`sessionStorage`). Reloads within the tab, such as switching country, don't show it. Tap to skip.
- **In tests:** automated test browsers skip it unless the address has `?splash`.

### Defaults chosen (easy to change)
- Medals apply to the sightings achievements too.
- "Every life stage" for Life Cycle Watcher means the larva, the pupa and **any one** adult form.
- Search results from other countries can't open a full card, because that country's notes aren't loaded.
- The splash tagline is a first draft.

## EPUB Library and reader

Bookworm can keep a Library of EPUB e-books and open them in its own reader, which counts the pages you actually read.

### Library

- **Where:** a Library card on the Reading screen, below Want To Read. **Add EPUBs** picks files; **Add a folder** imports every `.epub` in a folder (shown where the browser supports folder picking). Folder import is a one-off copy, not a live link to the folder.
- **Storage:** books are kept in this browser's IndexedDB (database `bookworm-library`: `meta` holds each book's details and reading record, `files` holds the EPUB). The Library is shared by every country. It isn't included in Export Save, which would make save files huge. Bookworm asks the browser to keep this storage (`navigator.storage.persist`).
- **DRM:** only DRM-free EPUBs work. A book with `META-INF/rights.xml`, or anything in `META-INF/encryption.xml` other than font obfuscation, is refused with a message naming the file. Duplicates (same title, author and file size) are skipped.
- **Rows:** title, author, page count and either "Not started", the % read, or the larva the book is feeding. **Read** opens the reader. The full list (See the whole Library) also has **Start a larva** and **Remove** (tap twice).

### Pages

- Each section of the book is split into **pages of about 250 words** (1,250 non-space characters, `PIECE`), so counts don't match a printed copy. An invisible marker goes where each page starts, at a word boundary.
- **Only the main text counts.** Left out: the cover, title page, contents, copyright page, dedication, index, glossary, notes, bibliography, acknowledgements, appendices and other front and back matter, plus spine items marked `linear="no"` and image-only pages. These are found from EPUB 3 landmarks and `epub:type`, EPUB 2 `<guide>` entries, or (only when a book has neither) file names such as `index.xhtml` or `glossary.html`. Calibre-style names like `index_split_001.html` aren't caught.
- A larva started from the Library uses the book's main page count. A book already being read (with its own printed page count) can be **linked** instead, and reader pages are scaled to its pages.

### The reader

- One section at a time, laid out in CSS columns, one screen per column. Previous and next buttons, tapping the left or right third of the page, swiping, and arrow keys all turn pages. Contents (from the EPUB 3 nav or EPUB 2 NCX), text size buttons, and a **Back** button after following a link inside the book.
- Book styles and scripts are removed and Bookworm's own typography is used, so nothing in a book can run. Images are shown from the EPUB; outside links open in a new tab.
- It opens at the first main section, then remembers your place, even after a text-size change or a rotation.
- Written from scratch with no outside libraries: a small ZIP reader using the browser's `DecompressionStream`, then `container.xml` → OPF → nav or NCX.

### Automatic tracking (Settings → Epub Reader, on by default)

- **What counts:** every second the reader is on screen, each page of text on the current screen gets time (shared out if it spans several screens). A page is read once it has had long enough to read it: its length at 80 characters a second (`READ_CPS`, about 800 words a minute, a fast skim), and at least 3 seconds. That's about 15 to 20 seconds for a full page.
- **What doesn't:** flicking past pages, time in the background (the timer pauses when the tab is hidden, and gaps of more than 5 seconds are capped), time with the contents open, and rereading. Jumping to the index or glossary and back doesn't count the pages in between, and the index and glossary themselves never count.
- **Nothing is fed automatically.** Closing the reader shows a summary: new pages this session (and about how many words), any earlier pages not yet fed, and the % of the book read. It offers **Feed** (or **Morph** for a pupa), **Log feeding manually instead** (marks those pages as handled and puts the cursor in the book's log box) and **Not now**. It also says that automatic tracking can be turned off in Settings → Epub Reader.
- Each linked book remembers how many reader pages have been fed (`libFed`), so pages are never fed twice. Unfed pages also show on the book's page, with a Feed button.
- **Finishing:** once the last main page and at least 90% of the main pages have been read (`FINISH_SHARE`), feeding finishes the book, so a skipped page or two doesn't stop a larva emerging.
- An unlinked book's summary offers **Start a larva with this book**; the pages already read are then ready to feed. Linking an EPUB to a book already in progress doesn't feed pages read before the link.
- With tracking off, the reader still remembers your place, but no pages are counted and no summary is shown.

## Species cards

Each card shows:
- name, category and scientific name;
- rarity chip and "In season now!" chip;
- the I've Seen It! box;
- family, size, larva food, when to see it, and UK or Irish status.

Then, in order:

1. **Field notes.** These unlock once you've raised the species from a book.
2. **Gardening Tip.** It only appears for species a garden can realistically attract, for example: *"Want to attract a Red Admiral to your garden? Consider adding stinging nettles and ivy."* There's a tip for 73 British and 64 Irish species. There's **no** tip for:
   - garden pests, such as cabbage whites, rosemary beetle and box-tree moth;
   - anything that bites, stings or has irritating hairs;
   - rare habitat specialists;
   - predators and ants.

   No invasive plants are suggested, and buddleia was deliberately left out.
3. **Want To Learn More?** Free guides from non-commercial organisations: the Wildlife Trusts, Butterfly Conservation, the Bumblebee Conservation Trust and, in Irish mode, the National Biodiversity Data Centre, Butterfly Conservation Ireland and the All-Ireland Pollinator Plan. Every link was checked before it was added. Butterfly Conservation has no page for a handful of moths, so those cards don't link there.
4. **Additional Sources.** Every page the card's facts were checked against, including Wikipedia, plus a note saying when the check was done. If a detail couldn't be confirmed by any source, the card says so.
5. **Raised from your books.** Which books raised this species.

The gardening tip and links show even before a species is raised.

## Fact-checking and sources

In October 2026 every card was checked claim by claim against trustworthy sources. These were mainly:
- conservation charities (Wildlife Trusts, Butterfly Conservation, Bumblebee Conservation Trust, Buglife, Woodland Trust, RHS, Natural History Museum);
- recording schemes (BWARS, UKMoths, UK Beetle Recording, sawflies.org.uk, NatureSpot);
- Irish public bodies (NBDC, NPWS, All-Ireland Pollinator Plan, Habitas);
- Wikipedia.

Retailers, pest-control firms, blogs and forums were not allowed.

| Result | Field-note sentences | Other details (food, season, size, status, scientific name) |
|---|---|---|
| Confirmed as written | 455 | 833 |
| Corrected | 220 | 344 |
| Replaced with a verified fact | 38 | — |
| Kept but unconfirmed (flagged on the card) | — | 15 |

**Notable corrections:**
- **Scientific names:** Adonis and Chalk Hill Blues are now *Polyommatus*, the Gooseberry Sawfly is *Euura ribesii*, and the Brown Lacewing now uses its family name, Hemerobiidae.
- **Irish wood ants:** Ireland has **two** wood ants. As well as the Hairy Wood Ant, the Scottish Wood Ant lives at two sites in County Armagh.
- **Mullein:** rediscovered in Ireland in **2021** after 69 years.
- **Privet Hawk-moth:** Britain's largest resident *hawk-moth*, not its largest moth.

**Where sources disagree** (for example Scotch Argus wingspan and Narrow-headed Ant size), the national recording scheme or Butterfly Conservation figure was used.

The raw fact-check results are in `fc/result*.json`. For every claim they record the status (ok, corrected, replaced or unverified), the old wording and the URLs used. The checkers' notes on disputed points are in those files too.

### Fact-checking American species

- **Rules:** `fc/US_INSTRUCTIONS.md`. The allowed sources are US university extension services and entomology departments, USDA, the US Forest Service, US Fish & Wildlife Service, the National Park Service, state natural resources departments, the Smithsonian, the Xerces Society, NABA, BugGuide, Butterflies and Moths of North America, AntWiki and Discover Life, then Wikipedia. Pest-control companies, retailers, blogs and forums are never used.
- **Order:** smallest category first: Ants, Other insects, Flies, Bees & Wasps, Beetles, Moths, Butterflies.
- **How results get in:** `python3 fc/merge_us.py fc/us_result_<category>.json [fc/us_over_<category>.json]` adds a category's results to `fc/us_checked_all.json` and rewrites the `US_CHECKED` block in `bookworm.html`. The optional overrides file holds small wording edits made after checking (for example "sidewalks" instead of "pavements", or feet before metres).
- **What `US_CHECKED` holds:**
  - `species`: full checked cards for American-only species, copied into `CHECKED`;
  - `caste` and `dimorph`: checked queen/worker and male/female descriptions, with sources;
  - `notes`: the checked American status and note for species shared with the UK.
- **On the card:** checked American cards list their sources and say "Every field note on this card was checked…". Shared species say whether the American status and note were checked too. Anything no source confirmed is named as a best guess.
- **Ants (October 2026):** 92 fields: 63 ok, 23 corrected, 0 replaced, 6 unverified.
  - **Unverified:** the adult season for six ants. Sources give mating-flight months but not when workers are about.
  - **Notable corrections:**
    - The Texas Leafcutter Ant can have several queens, not one.
    - The Allegheny Mound Ant hibernates from mid-October to mid-May, so its season is now May–October.
    - Worker sizes: Honeypot Ant 3–7 mm, Odorous House Ant 1.5–3.3 mm and Red Harvester Ant 5–10 mm (sources disagree: 5–7 mm or about 10 mm).
    - Queen lengths with no source (fire ant, odorous house ant, Argentine ant) were replaced by "larger than workers".
    - The European Fire Ant is invasive in the Northwest as well as the Northeast.
    - The "most painful sting in North America" ranking for the Red Harvester Ant couldn't be confirmed, so the card now says the sting is very painful.
  - **Tiny sizes in inches:** sizes under half an inch now keep two decimals, e.g. "0.06–0.13 in (1.5–3.3 mm)", so small ranges no longer collapse to "0.1 in". If a range starts under half an inch, both ends get two decimals ("0.31–0.59 in").
- **Other insects (October 2026):** 88 fields: 54 ok, 31 corrected, 0 replaced, 3 unverified.
  - **Unverified:**
    - the Owlfly and Snakefly seasons;
    - the Hanging Scorpionfly's larval food.
  - **Sizes:** most body lengths were too large. Dobsonfly 48–60 mm, Wasp Mantidfly 23–30 mm, Earwigfly 8–15 mm and Hanging Scorpionfly about 19 mm. The Fishfly, Owlfly and Snakefly sizes come from genus or family figures.
  - **Names:** the Hanging Scorpionfly is now *Hylobittacus apicalis*. BugGuide now files the Antlion as *Neleon immaculatus*, but *Myrmeleon* is kept because Wikipedia and Ohio State still use it.
  - **Other corrections:**
    - Earwigfly larvae have never been found, not "hardly ever".
    - Both sexes of the Summer Fishfly have comb-like antennae.
    - The Snakefly's food now describes the larva's diet, not the adult's.
  - **Shared species that are different species in the US:** four of the shared "Other insects" are European species whose American relatives are different. In the US their scientific name shows the genus instead (`sci` in `US_CHECKED.notes`), and the American note says North America has its own kinds:
    - Common Green Lacewing → *Chrysoperla* species;
    - Scorpionfly → *Panorpa* species;
    - Alder Fly → *Sialis* species (North America has about 22);
    - Snow Flea → *Boreus* species.
  - **Snow Scorpionfly:** in the US, "snow flea" means a springtail, so the Snow Flea is called the **Snow Scorpionfly** there. Its family label is "Boreidae (snow scorpionflies)".
  - **BugGuide searches:** these now skip words like "species" and "and relatives", so they search for the genus or family.
- **Flies (October 2026):** 155 fields: 87 ok, 53 corrected, 4 replaced, 11 unverified.
  - **Unverified:**
    - seven seasons;
    - the sizes of the Greenhead, Long-legged Fly and Rabbit Bot Fly;
    - the Transverse Flower Fly's larval food. Its larva has never been described.
  - **Rabbit Bot Fly:** the card is now *Cuterebra* species, "found across most of North America" (BugGuide). *Cuterebra cuniculi* itself lives only in Georgia and Florida.
  - **Larval food:** two cards had it wrong. Deer Fly larvae eat rotting plant matter, not small animals. The Long-legged Fly's food line described the adults.
  - **Replaced facts:**
    - The American Hover Fly's hovering fact is now its fall migration south from Canada.
    - The Black Horse Fly's "one of the largest" claim is now "black all over, even its wings".
    - The Golden-backed Snipe Fly's head-down resting pose is now that adults visit elderberry flowers.
    - The Transverse Flower Fly's rat-tailed maggot fact is now its range.
  - **Corrections worth knowing:**
    - The Transverse Flower Fly's yellow is a patch at the back of the thorax, not a band.
    - Black Soldier Fly adults "eat little, if anything".
    - Fruit fly eyes are brick-red.
    - The Asian Tiger Mosquito is 2–10 mm.
  - **Shared species:** all four (Drone-fly, Crane Fly, Narcissus Bulb Fly and Dark-edged Bee-fly) really live in the US, so none needed a `us_sci`.
    - The Crane Fly's American status is now "Introduced; a lawn pest in the Northwest and Northeast".
    - Its note now says North America has more than 1,600 kinds.
    - Its American "daddy longlegs" sentence now says the name "is also used for" harvestmen and cellar spiders. No source said that use is more common in the US.
  - **Sentences shown only in the US:** these (`COUNTRY_TEXT`) are now fact-checked too. Their sources are stored as `extra` on the species' `US_CHECKED.notes` entry.
  - **Overrides:** the overrides file can now set `sci` and add `src`, used here for the Rabbit Bot Fly.
- **Bees & Wasps (October 2026):** 260 fields: 194 ok, 47 corrected, 1 replaced, 18 unverified. Most of the unverified ones are seasons, plus three sizes.
  - **Bicolored Striped Sweat Bee:** the female isn't green all over. She has a green head and thorax and a black-and-white striped abdomen.
  - **Black-and-yellow Mud Dauber:** the "organ pipe" nest belongs to a different wasp. This one builds a smooth lump of mud cells.
  - **Rusty Patched Bumblebee:** it lives in 13 states and Ontario. It was the first bumblebee listed as endangered in the continental US.
  - **Eastern Velvet Ant:** sources disagree on the larval food (bumblebee nests, or cicada killers and other ground-nesting wasps), so the card mentions both.
  - **Squash Bee:** it collects *pollen* only from squash, pumpkins and gourds, early in the morning.
  - **Sizes:** several were corrected, e.g. Common Eastern Bumblebee 8–23 mm, Cicada Killer 15–50 mm, Blue Orchard Bee 9–11 mm and Pigeon Tremex 20–30 mm.
  - **Queen and worker descriptions:** checked and corrected where needed, e.g. sizes and paper wasp colours.
  - **Shared species:** Honey Bee, European Hornet and Wool-carder Bee are the same species in the US, so none needed a `us_sci`.
    - The European Hornet fact "the only true hornet established here" still holds. The Yellow-legged Hornet found in Georgia in 2023 isn't established and is being eradicated. **Recheck this one in future.**
    - The British fact comparing the Hornet with the Common Wasp, which isn't on the US list, now reads "It's usually less defensive than yellowjackets or bald-faced hornets" in the US (Clemson HGIC).
  - **Gardening tips:** these are now part of the American check, and five changed:
    - American Bumblebee: "sunflowers and goldenrods". Red clover isn't native.
    - Eastern Carpenter Bee: "salvias and passionflower (maypop)".
    - Bicolored Striped Sweat Bee: "coneflowers and goldenrods".
    - Alfalfa Leafcutter Bee: "alfalfa".
    - Feather-legged Fly: "dill and native asters". Fennel is invasive in California.

    The other two fly tips (American Hover Fly, Transverse Flower Fly) were confirmed.
  - **Bug fixed:** the first American checks had dropped the Gardening Tip from three flies. Checked American cards now keep their draft tip unless the check changes it (`merge_us.py` stores `garden`).
  - **Bug fixed:** the British-to-American name swap could double a name ("European European Hornet"). American names already in the text are now protected first.
- **Beetles (October 2026):** 315 fields: 219 ok, 79 corrected, 5 replaced, 12 unverified. Most of the unverified ones are seasons.
  - **Sizes:** many were too narrow. Big Dipper Firefly 9–19 mm, Synchronous Firefly 11–15 mm, American Burying Beetle 25–45 mm.
  - **Overstated facts, now toned down:**
    - The Emerald Ash Borer has killed "tens of millions" of ash trees, not hundreds of millions.
    - The Hercules Beetle is one of the heaviest *beetles*, not insects.
    - The Boll Weevil has been wiped out everywhere except a small part of South Texas.
  - **Male/female forms:** female Hercules Beetles have no horns at all.
  - **Replaced facts:**
    - the tiger beetle stopping to see its prey again;
    - the bombardier beetle aiming its spray (that was shown for African species);
    - the Dogbane Leaf Beetle playing dead;
    - two Pleasing Fungus Beetle facts.
  - **Bess Beetle season:** April–August, when adults are out and about (BugGuide: they come to lights in spring and summer). A new field note says they can be found in rotting logs all year round.
  - **Season rule:** "When to see it" means when adults are **out and about**. If they can also be found hiding all year, that goes in a field note. `US_INSTRUCTIONS.md` now says so. Extra facts can be appended with an overrides file (`"facts": {"3": "..."}`).
  - **Shared species:** all five are the same species in the US, so none needed a `us_sci`. The Devil's Coach Horse is established on the West Coast.
    - The Harlequin Ladybird's American name is now **Multicolored Asian Lady Beetle** (University of Maine Extension, BugGuide), set with `fc/us_over_beetle.json`.
    - The 7-spot note now says that aphid-control releases failed and that the wild population probably arrived by accident in the 1970s.
    - The 14-spot arrived by accident near Quebec in the 1960s and is widespread in the East.
  - **Gardening tips:** all three confirmed (native milkweeds, goldenrod, dill and yarrow).
- **Moths (October 2026):** checked in two halves at the same time. 463 fields: 290 ok, 162 corrected, 7 replaced, 4 unverified. Most corrections bring wingspans and flight seasons into line with Butterflies and Moths of North America. Southern broods often make the season longer.
  - **Wrong claims fixed:**
    - Milkweed Tussock Moth caterpillars use older milkweed, which Monarchs avoid; they don't feed "alongside" them.
    - The Hag Moth's monkey slug has nine pairs of arms, not six.
    - In Mexican folklore the Black Witch is an omen of death. The "money moth" belief comes from the Bahamas.
    - The Fall Webworm's spread is confirmed for Europe only.
    - "Like a sweet" is now "like a piece of candy" (Rosy Maple Moth).
  - **Replaced facts** (claims that couldn't be confirmed):
    - Carolina Sphinx tongue length;
    - catalpa planting by anglers;
    - Big Poplar Sphinx size ranking;
    - Tersa Sphinx body shape;
    - Achemon Sphinx losing its horn;
    - Sheep Moth spines stinging;
    - Faithful Beauty "flies slowly". It now oozes yellow foam to put off predators.
  - **Scientific name:** the Squash Vine Borer is now *Eichlinia cucurbitae*. Its Butterflies and Moths of North America page is still under *Melittia cucurbitae*, so `BAMONA_SLUG` points there. The Eichlinia address gives a 404.
  - **Links checked:** the Fall Webworm and Virginian Tiger Moth pages on Butterflies and Moths of North America both exist.
  - **Shared species:** all eight are the same species in the US, so none needed a `us_sci`.
    - The Garden Tiger is called the **Great Tiger Moth** in the US (BugGuide), set with `fc/us_over_moth2.json`.
    - The Peppered Moth keeps its name, because it's the name used in American textbooks. Its American note now mentions its other US name, Pepper-and-salt Geometer.
    - The Cinnabar was first released in California in 1959, to control tansy ragwort.
    - The Box-tree Moth was first found in New York in 2021 and has since spread to many eastern and Great Lakes states (APHIS).
    - The Large Yellow Underwing arrived in Nova Scotia in 1979.
  - **Gardening tips:**
    - Hummingbird Clearwing: "native coral honeysuckle and bee balm". The old honeysuckle and viburnum hosts on record include invasive species.
    - Yucca Moth: "native yuccas such as Adam's needle".
    - Clymene Moth's larval food is now bonesets, white snakeroot, oaks and willows.
  - **Could be added later:** the official US spelling "Indianmeal Moth".
- **Butterflies (October 2026):** checked in two halves at the same time. 595 fields: 402 ok, 180 corrected, 9 replaced, 4 unverified.
  - **Corrections:** most are wingspans (often too narrow, e.g. Giant Swallowtail 117–188 mm) and flight seasons. Many southern species now fly longer, or all year in the Deep South.
  - **Wrong claims fixed:**
    - The Eastern Tiger Swallowtail is Virginia's state *insect*. It's the state butterfly of Georgia, Delaware, Alabama and the Carolinas.
    - Cloudless Sulphur caterpillars don't build leaf tents.
    - Mourning Cloak: "Camberwell Beauty" is the British name.
  - **Status updates:**
    - Monarch: proposed as federally threatened in December 2024; the final decision is due fall 2026 at the earliest.
    - Regal Fritillary: proposed for protection in 2024; in the East it survives at one site in Pennsylvania.
    - Miami Blue: the last confirmed wild adult was seen in July 2022 (US Fish & Wildlife Service 2024 review).
    - Atala: spreading north with coontie gardens.
  - **Scientific names:**
    - Schaus' Swallowtail: *Heraclides aristodemus ponceanus*.
    - Karner Blue: *Lycaeides melissa samuelis*, its federal listing name.
    - Miami Blue: *Cyclargus thomasi bethunebakeri*.
    - Common Checkered-Skipper: *Burnsius communis*.
    - Zabulon Skipper: *Lon zabulon*.

    The Butterflies and Moths of North America pages for the last three changes are still under their old names, so `BAMONA_SLUG` pins them. The Karner Blue was already pinned.
  - **Gardening tips:** fennel was dropped everywhere (invasive in California).
    - Black Swallowtail: dill, parsley and golden Alexanders.
    - Anise Swallowtail: native biscuitroots and other carrot-family herbs.
    - Spring Azure: flowering dogwood and New Jersey tea.
    - West Coast Lady: native mallows or hollyhocks.
    - Silver-spotted Skipper: false indigo or native American wisteria, plus blazing star.
  - **Shared butterflies:** all five are the same species in the US. Their British field notes were also checked for American readers (`uk_facts_shown_in_us`). The rewrites are stored as `swap` pairs on `US_CHECKED.notes` and are swapped in before British place names are filtered out.
    - **Monarch:** one distance, "about 3,000 miles (4,800 km)", from the US Fish & Wildlife Service and the US Forest Service. Other new notes: the Western Monarchs' shorter trip to the California coast, the Mexican fir-forest roosts, and corrected toxin and scent-patch wording.
    - **Painted Lady:** the North American migration from northern Mexico to Canada and Alaska replaces the Africa to Arctic one. It also gets the 2017 Denver radar swarm and the northern US winter.
    - **Red Admiral:** spring recolonization from the South, winter survival only in the South, and tree sap and fermenting fruit.
    - **American Copper:** grey hindwing with an orange-red edge. It's thought to spend the winter as a chrysalis in North America.
  - **Checker suggestions** for the design notes (Aphrodite and Northern Crescent counting as the same collectible) couldn't be confirmed from the pages opened, so they're marked unverified.
  - **A few sources are outside the preferred list:**
    - Miami Blue review copy on miamiblue.org;
    - Viceroy as Kentucky's state butterfly (Bernheim Arboretum);
    - Giant Swallowtail spreading north (bioRxiv preprint).
- **Units, improved:**
  - Distance ranges convert both ends, e.g. "300 to 990 miles (480 to 1,600 km)".
  - Heights in metres get feet: "7,900 to 11,800 ft (2,400 to 3,600 m)" in the US, and the other way round in the UK and Ireland.
  - Text that already has both units is put in the right order for the country.
- **Duplicate facts:** these are removed after conversion, so a rewritten fact and a winter note that say the same thing only show once.
- **Wording:**
  - In American English, true flies are two words, as in American field guides: "crane flies", "horse flies", "robber flies", "snipe flies", "soldier flies". Caddisflies and alderflies stay one word.
  - The "best guess" note now says "season" instead of "when to see it".
- **Instructions update:** for shared species, checkers now also confirm that the British species really lives in the US. If not, they give a `us_sci`, and the overrides file can rename the species for the US (`name`).

## Language and units

### Choosing a language
- **Language is separate from country.** There are two languages: **British English** and **American English**.
  - The language starts out matching the country: American English for the US, British English for the UK and Ireland.
  - It's stored in `bookworm-lang`, so it's included in Export/Import.
- **Prompt when switching country:** moving between the US and the UK or Ireland asks "Would you like to switch to American English?" (or British English).
  - **Yes** switches language and country together.
  - **No** keeps your current language. Moving between the UK and Ireland never asks.
- **Settings → Language:** changes it at any time. The page reloads, because all text is converted once at load.

### What follows what
| Follows the **language** | Follows the **country** |
|---|---|
| Spelling ("colour" or "color") | Which species there are |
| Everyday words: autumn/fall, pavement/sidewalk, ladybird/lady beetle, hoverfly/hover fly, kitchen cupboards/kitchen cabinets and pantries, rubbish/garbage, "your garden"/"your yard" | **Species names** (official British, Irish or American names, e.g. "7-spot Ladybird" in the UK, "Asian Lady Beetle" in the US) |
| Kind labels and family labels ("Lady beetle", "Coccinellidae (lady beetles)") | Habitat names ("Backyards, Towns & Homes") |
| The Gardening Tip and About wording | Facts that differ in the US, e.g. no wild hedgehogs, "daddy longlegs" means harvestmen (`COUNTRY_TEXT`) |
| | Units (see below) |

### How the conversion works
- **No duplicate text:** each piece of text is written once. UK and Irish data is in British English, and American-only data and `US_CHECKED` are in American English.
- **At load,** `usText()` runs `countryText()` (US facts and American names of shared species, US only) and then `langText()` (language). It runs on:
  - species food, season, status, gardening plants, nouns and facts;
  - queen, worker and male/female descriptions;
  - caste science and spring-queen notes;
  - winter notes;
  - growth-stage facts and pupa facts;
  - habitat descriptions.
- **Spelling:**
  - `SPELL_US` holds British → American stems with allowed endings ("colour" → "color", "moult" → "molt", "recognise" → "recognize", "travelling" → "traveling").
  - `SPELL_GB` is the same list run backwards. It leaves out the few that don't reverse safely (whilst, amongst, learnt, tyre, licence, programme), because "while" isn't always "whilst" and "tire" is also a verb.
- **Words:**
  - `US_WORDS` handles British → American.
  - `GB_WORDS` handles American → British. "Fall" only becomes "autumn" in season phrases ("in the fall", "each fall", "spring to fall"), never as a verb.
- **Names are protected:**
  - Every species name in the current country, the US habitat names and "Colorado" are masked while converting, so they're never changed. "Pavement Ant", "Clouded Sulphur" and "Gray Hairstreak" keep their official spellings.
  - Ids and URLs are never changed.
- **When it runs:** nothing is converted for the UK or Ireland in British English, because the text is already right.
- **Python copy:** `spell.py` and `spell_rules.json` hold the same British → American rules. `fc/merge_us.py` uses them to keep checked American text in American English.

### Units
- **US:** inches first with metric in brackets: "3.1–5.5 in (79–140 mm)".
- **UK and Ireland:** metric first with imperial in brackets, for readers who grew up with inches: "79–140 mm (3.1–5.5 in)".
- **Distances:** these work the same way: "2,800 miles (4,500 km)" in the US and "4,500 km (2,800 miles)" in the UK and Ireland. Grid names like "10 km map squares" are left alone.
- **Where it applies:** wingspans and body lengths, sizes in facts, form descriptions, the larva's length chip and the growth screens. See `withInches()`, `withMiles()` and `lenText()`.
- **Precision:** sizes under half an inch keep two decimals ("0.06–0.13 in"), and both ends of a range use the same precision.

### Checks
- `spell_test.js` opens every species card for five country/language pairs: US in American, UK in British, Ireland in British, US in British and UK in American. It confirms that no spelling or word from the other language appears outside species names.
- `lang_test.js` checks the switching prompt, Settings → Language, kept names, units, and that US-only facts stay when the US is read in British English.

## Book search

- **Search:** it uses the free **Open Library** API, through `/search.json`, `/isbn/<isbn>.json` and `/authors/<id>.json`.
- **Title capitalisation:** titles from Open Library are often in lower case. So when a result is shown or picked, they're put into publisher-style title case: every word's first letter is capitalised, including each part of a hyphenated word ("right-wing" becomes "Right-Wing"), except short articles, conjunctions and prepositions ("a", "and", "of", "the", "with" and so on; see `SMALL_WORDS`). Those stay lower-case unless they're the first or last word or follow a colon or dash, so you get "Harry Potter and the Philosopher's Stone", "Of Mice and Men" and "Run-of-the-Mill". All-capital words are left alone. Existing capitals are kept ("NASA", "McDonald"), and letters after an apostrophe aren't changed ("Don't"). You can still edit the title by hand.
- **Offline fallback:** when Open Library can't be reached, it falls back to a small built-in sample catalogue. The claude.ai artifact sandbox blocks outside requests, so live search only works in the GitHub Pages version.

## Project files

| File | What it is |
|---|---|
| `bookworm.html` | The app. It's one HTML file with inline CSS, JavaScript and SVG, and it's the source of truth. |
| `build_standalone.py` | Wraps `bookworm.html` in a full HTML document, with doctype, meta tags and favicon, and writes `index.html`. **Run `python3 build_standalone.py` after every change.** |
| `index.html` | The built page that goes to GitHub Pages. |
| `bookworm.v*.html` | Backups from earlier versions. |
| `fc/` | Fact-check batches (`batch*.json`), results (`result*.json`) and the checker instructions (`INSTRUCTIONS.md`). American ones are `us_batch_*.json`, `us_result_*.json`, `US_INSTRUCTIONS.md`, `merge_us.py` and the combined `us_checked_all.json`. |
| `art/`, `embed_art.py` | Real artwork (transparent WebP, one per species and stage), the clean-up script and its recipes; `embed_art.py` puts the images into `bookworm.html`. |
| `build_index.js` | Rebuilds `SEARCH_INDEX` (every country's species, for the collection search) inside `bookworm.html`. Run after changing species data. |
| `*_test.js` | Playwright test scripts. |
| `make_test_epubs.py`, `test_epubs/` | Builds three small test EPUBs: an EPUB 3 book (nav, landmarks, glossary, index, an image and an internal link), an EPUB 2 book (NCX and guide) and a "DRM" book. |

## How the code is organised

Everything runs inside one IIFE in `bookworm.html`.

### Species data
- **`SPECIES`:** the original 15 butterflies as object literals.
- **Category rows:** `BFLY_ROWS`, `MOTH_ROWS`, `BEEWASP_ROWS`, `BEETLE_ROWS`, `ANT_ROWS`, `FLY_ROWS` and `OTHER_ROWS`. Each row is laid out as `[id, name, sci, family, rarity, size, larvaLength, food, season, status, facts, noun]`. `MOTH_ROWS` rows have no `noun`.
- **Extras:** `EXTRA` holds species added for both countries, and `IE_ONLY` holds Ireland-only species.
- **`HAB`:** each species' habitats and sources. `HABITATS` lists the 8 habitats with descriptions. Unlocks are stored in `state.habitats`, using `unlockHabitatFor()` and `replayHabitats()`.
- **`DUR`:** active larva and pupa days, basis (species, genus or family), winter note and sources. `pupAt(id)` gives the pupation %, `stageAt(k,id)` the % for each stage, and `stageFor(p,id)` the stage for a given %.
- **`DIMORPH`:** the species with male and female forms, as `[male description, female description, sources]`.
- **`CASTE`:** the queen, worker and (Red-tailed Bumblebee) male descriptions with sources, plus `why` (how caste is decided, for ants, honey bees, bumblebees and wasps) and `spring` (spring-queen field notes).
- **Forms in code:** `formsOf(id)` lists a species' forms: `m` male, `f` female, `q` queen, `w` worker. Larvae store `sex` (`m` or `f`), and `finalForm()` turns a female of a social species into `q` or `w` when she emerges. Whether she ever went hungry is tracked by `markHungry()`.
- **`CHECKED`:** the fact-checked values, which **override** the draft text above. It holds `sci`, `food`, `season`, `size`, `status`, `facts`, Irish `[status, fact]`, `garden`, `src`, `srcIE` and `unv`. See the tidy-up note in [Before the Android build](#before-the-android-build).

### Category metadata
`KINDS` holds per-category wording: larva and pupa words, size and food labels, and stage facts.

### Seasons
`parseMonths()` turns "When to see it" text into a set of months. It understands:
- month ranges, including ranges that wrap the year;
- "All year";
- season words such as "spring" or "summer".

`inSeason()` and `weightedPick()` handle the 3:1 seasonal weighting.

### Countries
`COUNTRIES`, `IE_RARITY`, `IE_NOTES`, `IE_STATUS`, `PLACE_RE` and `COUNTRY_IDS`. The species list is filtered to the chosen country at load.

### Card links and sources
`learnLinks()`, `sourceLinks()`, `srcLabel()` (turns URLs into readable labels) and `renderLearn()`.

### Main screens
`renderShelf`, `renderRead`, `feed`, `celebrate`, `renderColl`, `openSpecies`, `openSeen`, `openWaiting`, `openNotes`, `openWtrList`, `openDoneList`, `pickSpecies` (with `missingForms`), `checkAchievements`, `checkUnlocks` and `renderAch`.

### Save data
Each country's save holds:
- `books`: being read or frozen, each with an optional `sex`.
- `waiting`: larvae waiting for a book.
- `collection`: every finished book with the species, form and `country` it was raised in. This doubles as the Completed Books list.
- Merge Collections is stored separately in `bookworm-merged`, because it applies to every country.
- `wtr`: the Want To Read list.
- `seen`: the I've Seen It! ticks.
- `achievements`.
- `unlocks`: when each category unlocked.

### Art
The art is **placeholder** inline SVG (`caterpillarSVG`, `chrysalisSVG`, `butterflySVG`), to be replaced with your own artwork.

### Theme
Colour tokens are defined in `:root`, with light and dark mode. Fonts are Grandstander (display), Atkinson Hyperlegible (body) and DM Mono.

## Testing

The Playwright scripts run with `NODE_PATH=$(npm root -g) node <script>.js`. Outside network requests are blocked in tests.

- **Current:** `notes_test.js` covers:
  - butterfly-only start, and the Moths unlock after 2 butterflies;
  - See Field Notes;
  - Log Reading position;
  - the duplicate-book prompt;
  - Want To Read and Completed Books;
  - the two-line collection count;
  - locked-category notes.

  `caste_test.js` covers queens and workers: the feeding rule, the notes and the save migration.

  `multi_test.js` covers the shelf limits, freezing and transfers.
- **Also current:** `fc_test.js` checks every card in both countries:
  - no errors;
  - every season parses to months;
  - gardening tips show only where intended;
  - every card has sources.
- **New:**
  - `ui_test.js`: dates, favourites, Completed Books to species card, Want To Read reordering, About, sources and the fed-today heart;
  - `medal_test.js`: medal stages and the new achievements;
  - `transfer_test.js` and `transfer2_test.js`: taking books to another country and back, freezing, crediting home and release;
  - `index_test.js`: the search index is up to date, and search works;
  - `spell_test.js` and `lang_test.js`: language;
  - `epub_test.js`: importing (including DRM and duplicates), which sections count, the reader, dwell tracking, links and Back, text size, the close summary (Feed, Log feeding manually instead, start a larva), finishing a book, and the Settings switch. It uses Playwright's clock to fast-forward reading time.
- **Outdated:** several older scripts (`ie_test.js`, `merge_test.js`, `saw_test.js`, `meta_test.js`) still expect earlier species counts and wording, so they report stale failures. Update them during the tidy-up.

## Before the Android build

**A tidy-up pass is needed before porting.**

1. **Fold `CHECKED` into the species data.** The verified text currently sits in the `CHECKED` block, which overrides the original draft text at load. So the file holds both versions, roughly doubling the data size.
   - Write the checked values back into one clean dataset and delete the draft text. Ideally that's a single JSON file, `species.json`, with one record per species and a per-country section.
   - Drop the override step.
2. **Unify the data format.** Replace the mix of object literals and row arrays with one consistent record shape. Include rarity, Irish data, garden tip, sources and the unverified flags.
3. **Move `IE_RARITY`, `IE_NOTES` and the source maps** (`WT_SLUG`, `BC_SLUG`, `NBDC_ID` and so on) into the same dataset.
4. **Update or retire the outdated tests**, and add a check that every link still works.
5. **Re-check the 15 unconfirmed details**, flagged in `unv` and on the cards.
6. **Replace placeholder art**: a larva, a pupa and an adult for each category, or for each species.
7. **Swap `localStorage` for proper app storage** on Android, and keep the save-migration map (`MERGED`).
8. **EPUB folders on Android:** use the Storage Access Framework (a persistent folder permission) so the Library can watch a chosen folder for new books, instead of a one-off import. Move the Library from IndexedDB to app storage.

## Placeholder artwork

The monarch artwork used everywhere has been replaced by **temporary placeholder images**. They're greyscale, generic, and stamped "TEMP" across the middle. Real artwork can replace them one at a time later. They're drawn as inline SVG in the `art` section of the page, and `artFor(stage, mood, speciesId)` picks one.

| Stage | Image | Used for |
|---|---|---|
| Larva (stages 0–8) | Plain segmented larva that grows with each stage. It keeps the mood face and the hungry thought bubble ("!" when extremely hungry). | Every species |
| Resting stage (stage 9) | Hanging chrysalis | Butterflies |
| | Silk cocoon on a twig | Moths |
| | Barrel-shaped puparium in the soil | Flies. Most flies pupate inside their hardened last larval skin, which looks nothing like a cocoon. |
| | Pale pupa with legs, wings and antennae folded against the body | Bees, wasps, ants, beetles and other insects |
| Adult (stage 10, collection, species cards) | Butterfly | Butterflies |
| | Moth (furry body, feathery antennae, triangular forewings) | Moths |
| | Ant (side view) | Ants |
| | Wasp | Wasps, sawflies and other non-bee Hymenoptera |
| | Bee | Bees and bumblebees (`isBee()`) |
| | Beetle | Beetles |
| | Fly | Flies |
| | Flea (side view, wingless, big jumping legs) | Fleas (family Pulicidae, so the Snow Flea, a scorpionfly, isn't one) |
| | Lacewing-like insect | Other insects |

- **Where it shows:** the collection grid, the species cards, the male/female and queen/worker form art, the reading screen, the shelf and the celebration screens.
- **Uncollected species:** these still show as a faint silhouette.
- **Possible extra placeholders later:**
  - wingless adults such as worker ants and female winter moths;
  - a caddisfly larva in its case;
  - an antlion's sand pit.

## Real artwork

Placeholder art is being replaced species by species with hand-drawn artwork. The first is the **Red Admiral** by Tara Carter: a coloured-pencil adult, plus pencil sketches of the caterpillar and chrysalis.

- **Files:** `art/<species-id>-adult.webp`, `-larva.webp` and `-pupa.webp` are transparent WebP images.
  - `python3 embed_art.py` embeds them into `bookworm.html` (the `ART_DATA` block) as data URIs.
  - `ART_INFO` records where the adult's body is (`bodyX`, as a share of the width) and who drew it.
  - The species card shows "Artwork by …" under collected species with real art.
- **Cleaning up a photographed drawing:** `art/clean_art.py` does this automatically. The commands used for the Red Admiral are in `art/README.txt`.
  1. **Even out the paper:** it estimates the paper, including lighting and shadows, and divides it out, so the page becomes clean white and colours stay true.
  2. **Find the outline:** strokes are joined with a round brush, holes filled, specks and frame lines dropped. Inside the outline the art is opaque, so white wing spots stay white on a dark screen. Thin parts (antennae, spines) stay as soft pencil lines.
  3. **Tidy the tones:** levels make the darkest pencil near-black, then a gamma curve, a gentle colour boost and softened paper specks.
  4. **Sketches:** darker lines and whiter paper, plus an optional soft colour wash. The Red Admiral caterpillar and chrysalis have a light brown-grey wash.
- **How it's shown:**
  - **Adult:** drawn twice, clipped at the body line, so each half can flap in the existing wing animation.
  - **Caterpillar:** grows with each stage on the leaf, with the hungry thought bubble on top. It has no face, so the mood chip and bubble show hunger.
  - **Chrysalis:** hangs from the twig and sways.
  - **Uncollected species:** the art still shows as a silhouette.
- **Size:** about 290 KB for the three Red Admiral images. Roughly 650 species with three images each would make the page very large, so before the Android build, artwork should move into separate files rather than being embedded.
- **Photo tips:** even daylight, no shadow across the page, camera straight above, and a little space around each drawing.

## Ideas for later

1. **Species-specific growth-stage trivia.** At each growth stage, show larva and pupa trivia for that particular species where it's known, instead of the general larva facts. Use the species' own life-cycle data (`DUR`) in the blurbs that appear as the larva reaches each new growth stage. Examples:
   - "A Peacock caterpillar feeds for about 27 days before it forms its chrysalis."
   - "Stag Beetle larvae can spend years growing in rotting wood."
   - The species' winter note, shown at the right stage, e.g. on reaching the pupa stage for species that overwinter as pupae.
2. **Habitat artwork.** The basic Habitats screens, a title and a species list, are in. Later, each habitat becomes a piece of background art, with the adult art of every collected species that lives there overlaid on it.
   - **Time of day:** the art comes in **dawn, day, dusk and night** versions, and the habitat follows the real time. Only species active at that time of day appear: for example, butterflies by day and most moths at night, which needs activity-time data per species.
   - **Seasons:** each habitat also has **spring, summer, autumn and winter** versions, following the real date. Only species out at that time of year appear, using the existing `months` data.
   - **Viewing other times:** you can switch to view another time of day or season, but the habitat always opens on the current real time and season.

## Design decisions log

- **Collection order:** collection pages are ordered from Common to Rare.
- **Achievements:** achievements are per category and per country, with meta achievements across both countries.
- **Seasonal weighting:** seasonal species are favoured, not exclusive. Out-of-season species can still hatch.
- **Want To Learn More? links:** the National Trust is a charity, not a public body, and has no species pages. So "Want To Learn More?" uses non-commercial conservation charities and public bodies instead, and says so on the card.
- **NBN Atlas:** left out, because its pages couldn't be verified.
- **Gardening tips:** shown before raising, alongside the links, because they're useful straight away.
- **Unlock order:** after Bees & Wasps, the order is Beetles, Flies, Ants, then Other insects, two butterflies apart each time. Change `UNLOCKS` to adjust.
- **Male and female forms:** only one larva of a species can be raised at a time, even when the other form is still missing.
- **EPUB tracking:** feeding from the reader is offered, never automatic, and counts pages actually viewed rather than the farthest point reached, so jumping to the back of a book and returning doesn't count everything in between. Built without outside libraries (the package registry was blocked, and this keeps it working offline).
- **Queens and workers:** caste follows feeding rather than chance, which rewards steady reading and mirrors the real biology. The Common Wasp and Hornet have forms too, even though few people will ever tell their queens from workers in the wild, because collecting is the fun.

---

## Copyright

Copyright © 2026 Tara Carter. All rights reserved.

This repository is not currently released under an open-source license. You're welcome to view the code and try the app. Please don't copy, modify or redistribute it, or its text, data or artwork, without permission.
