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
- [Species cards](#species-cards)
- [Fact-checking and sources](#fact-checking-and-sources)
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
  - the Feed button reads **Grow**, and logging reading shows "Growing!" instead of "Munch!";
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
- **Where they are:** the Collection screen has a **Species / Habitats** switch. Each unlocked habitat opens a simple placeholder screen: its name and a list of the species that live there, listed in-season species first, then out-of-season ones, alphabetically within each group. Raised species are in bold and ticked. Species in season this month get the leaf mark, so you can see what should be active there now. Tapping a species name opens its species card, and closing the card returns to the habitat. Species cards also list their habitats as links: tapping one opens that habitat (or, if it's still locked, says how to unlock it).

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
- **Category achievements:** "Collect all British Butterflies", "Collect all Irish Moths" and so on, one per category per country.
- **Meta achievements:** "Collect All British Species" and "Collect All Irish Species". These read both countries' saves, and they're stored separately in `bookworm-meta`.

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

## Countries: United Kingdom, Ireland and the United States

- **Switching country.** Tap the flag in the top-right corner to change country. The page reloads with that country's data.
- **What's separate.** Each country has its own species list, rarity tiers, status lines, field notes, save and achievements. The saves are `bookworm-proto-v1`, `bookworm-proto-v1-ie` and `bookworm-proto-v1-us`, and the chosen country is stored in `bookworm-country`.
- **United States (in progress: everything except bees and wasps so far).** The collection has 214 species.
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
  - **Moth forms and unlocks:** six moths have male and female forms (Promethea, Io, Salt Marsh, Spongy, White-marked Tussock and Bagworm). In the US, moths unlock after 2 butterflies, as elsewhere. Their achievement is "Porch-Light Parade".
  - **How it was written:** the American species notes, rarity, sizes, seasons, life cycles, habitats, gardening tips and male/female forms were **written from general knowledge and haven't been fact-checked yet**, and species cards say so.
  - **Look-alikes:** these are folded in the same way as for the UK and Ireland, e.g. Canadian into Eastern Tiger Swallowtail, Eastern Comma into Question Mark, Northern into Pearl Crescent, Five-spotted Hawk Moth into Carolina Sphinx, Snowberry into Hummingbird Clearwing, and Forest into Eastern Tent Caterpillar Moth.
  - **What it has:** category achievements ("Star-Spangled Wings" for butterflies, "Porch-Light Parade" for moths) and a meta achievement, "Collect All American Species".
  - **Learn More links:** Butterflies and Moths of North America (each species page was checked), the North American Butterfly Association and the Xerces Society.
  - **Still to do:** fact-check the butterflies, moths and other insects, then add bees and wasps, the last category. They need queen and worker forms for bumblebees, yellowjackets and paper wasps. US-specific habitat descriptions are also needed, because the current eight are written for Britain and Ireland.
- **Coverage.** The **United Kingdom collection includes Northern Ireland**. Both countries get the complete experience.
- **Wording, chosen to respect cultural identity:**
  - The country picker reads **"United Kingdom"** and **"Ireland"**.
  - Species are described as **"British"** or **"Irish"**, as in "223 British species".
  - "UK" is fine inside field notes for brevity.
- **Irish mode details:**
  - Rarity comes from `IE_RARITY`.
  - Irish status lines and an optional Irish fact come from `IE_NOTES`. The default status lines are in `IE_STATUS`.
  - Facts that mention British places are hidden automatically by `PLACE_RE`.

## Settings

Tap the gear under the flag to open Settings.

- **About:** what the app does, its learning objectives, the version number (`APP_VERSION`, currently 0.9.0) and the GitHub Pages link. The wording is a first draft to edit.
- **Merge Collections (Easy Mode):** shares everything you've collected, past and future, across every country.
  - **How it works:** each raised adult records the country it was collected in (`country`). Nothing is copied. While merged, the app reads every country's saves together (`coll()`). So species raised in the United Kingdom that also live in Ireland count as collected in Ireland, which leaves only the rest, including Ireland-only species. This works for any country added later.
  - **What changes on screen:** the collection shows an "Easy Mode" chip. Species cards and Completed Books show where each one was collected. Category unlocks also count butterflies from every country.
  - **Unmerge Collections:** the menu item changes to this once merged. Unmerging goes back to each country's own collection. It removes any category achievements, meta achievements and category unlocks that were only complete because of merging. Nothing else depends on them, so nothing breaks.
- **Export / Import Save:** **Export** saves every `bookworm…` storage key to a dated `.json` file: every country's progress, the country choice, merge setting, meta achievements and titles. **Import** loads one, after a warning that it replaces everything in this browser. A copy-and-paste option covers browsers that can't download or pick files. Use it to move a tester to a new device, or to load a tester's save and reproduce a bug.
- **Reset App:** asks twice ("Are you sure? This cannot be undone."). In every country it then deletes:
  - collections and completed books;
  - achievements, meta achievements, titles, the score and category unlocks;
  - waiting larvae;
  - I've Seen It! ticks.

  It also switches Merge Collections off. A notice explains that the Want To Read list, frozen larvae and current books are kept and must be removed by hand.

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

## Book search

- **Search:** it uses the free **Open Library** API, through `/search.json`, `/isbn/<isbn>.json` and `/authors/<id>.json`.
- **Offline fallback:** when Open Library can't be reached, it falls back to a small built-in sample catalogue. The claude.ai artifact sandbox blocks outside requests, so live search only works in the GitHub Pages version.

## Project files

| File | What it is |
|---|---|
| `bookworm.html` | The app. It's one HTML file with inline CSS, JavaScript and SVG, and it's the source of truth. |
| `build_standalone.py` | Wraps `bookworm.html` in a full HTML document, with doctype, meta tags and favicon, and writes `index.html`. **Run `python3 build_standalone.py` after every change.** |
| `index.html` | The built page that goes to GitHub Pages. |
| `bookworm.v*.html` | Backups from earlier versions. |
| `fc/` | Fact-check batches (`batch*.json`), results (`result*.json`) and the checker instructions (`INSTRUCTIONS.md`). |
| `*_test.js` | Playwright test scripts. |

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
- **Queens and workers:** caste follows feeding rather than chance, which rewards steady reading and mirrors the real biology. The Common Wasp and Hornet have forms too, even though few people will ever tell their queens from workers in the wild, because collecting is the fun.

---

## Copyright

Copyright © 2026 Tara Carter. All rights reserved.

This repository is not currently released under an open-source license. You're welcome to view the code and try the app. Please don't copy, modify or redistribute it, or its text, data or artwork, without permission.
