# 🎧 Model Card: Music Recommender Simulation

## 1. Model Name  

**VibeFinder 1.0**

---

## 2. Intended Use  

VibeFinder suggests which songs from a small catalog a user will probably like. It does this by comparing a song's genre, mood, energy, and acoustic-ness to what the user says they want. It assumes the user already knows and can state their own taste (favorite genre, favorite mood, energy level, and whether they like acoustic songs). This is a classroom project, not a real production app. It is meant to show how a simple content-based recommender works, not to be used by real listeners.

**Not meant for:**
- Real-world music recommendations (the catalog is tiny, only 18 songs)
- Learning a user's taste automatically from listening habits (it only uses what the user types in, it has no memory or behavior tracking)
- Any claim of fairness or accuracy beyond a classroom demo

---

## 3. How the Model Works  

This recommender doesn't learn from what anyone else listened to. It just compares each song's own attributes to what the user says they want, and scores how well they line up. Four things matter: the song's genre, its mood, how energetic it is, and how acoustic it sounds. The user's taste profile says which genre and mood they like, roughly how much energy they want, and whether they tend to like acoustic songs.

To get a score, the song earns points for each thing that lines up: a bit of credit for matching genre, a bit more for matching mood (I decided mood should count for more, since two songs in the same genre can still feel completely different, while songs in different genres can share the same vibe), and a sliding amount of credit for energy depending on how close it is to what the user wants, not just whether it's high or low. There's also a small bonus or penalty depending on whether the song's acoustic-ness fits what the user usually likes. All of that gets added up into one number, and the song also gets a plain-English list of reasons for why it scored the way it did, so a user isn't just shown a number out of nowhere.

Once every song in the catalog has a score, the system just sorts them from highest to lowest and hands back the top handful. The starter code originally just returned the first few songs in the file with no real scoring at all, so the main change is that recommendations are now actually based on matching the user's stated preferences instead of file order.

---

## 4. Data  

The catalog is `data/songs.csv`. It has 18 songs. Each song has: title, artist, genre, mood, energy, tempo, valence, danceability, and acousticness.

I started with 10 songs and added 8 more, to cover genres and moods that weren't there yet (metal, country, r&b, latin, folk, electronic, blues, classical / angry, nostalgic, romantic, playful, melancholic, mysterious, sad, triumphant).

Most genres only have 1 song in the catalog. Only pop and lofi have 2. That means there's not much variety within most genres.

Things missing from the data: lyrics, artist popularity, release year, and any real listening history (likes, skips, play counts). The dataset only describes the song itself, not how people actually react to it.

---

## 5. Strengths  

It works best for users who give a clear, specific profile, like "pop, happy, high energy" or "lofi, chill, low energy." Those got clean, sensible top picks in testing (see Section 7).

The energy score is a strength. It rewards songs that are *close* to the target energy, not just "high energy" or "low energy." This matches how energy actually feels — a user who wants medium energy shouldn't get the loudest song in the catalog just because loud scores high.

Mood being weighted higher than genre also matched my own intuition. Songs in the same genre can feel very different (two pop songs, one happy and one intense), and songs in different genres can feel similar (two chill songs, one lofi and one ambient). Weighting mood higher captured that correctly in testing.

Every recommendation also comes with plain-English reasons, so a user can see *why* a song was picked instead of just a number.

---

## 6. Limitations and Bias 

One weakness I found while testing the "High-Energy Pop" profile is that the system has no idea an artist can repeat, so it can hand back a top-5 list that looks varied but isn't. In that test, `Sunrise City` landed at #1 and `Night Drive Loop` landed at #5 — both by the same artist, Neon Echo — just because `recommend_songs` sorts purely by score with no de-duplication step. A user skimming five different song titles would reasonably assume they're getting five different artists' worth of variety, when really they're only getting four. This gets worse as the catalog grows if one artist happens to have several songs that all score decently for a given taste, since nothing in the scoring or ranking logic ever checks for that kind of overlap. It's a real gap between what the system presents as "variety" and what it's actually giving the user.

---

## 7. Evaluation  

I tested 3 normal taste profiles and 5 adversarial/edge-case profiles against `recommend_songs`, and printed the top 5 results for each to see whether the scoring held up.

**High-Energy Pop** (`genre=pop, mood=happy, energy=0.85, likes_acoustic=False`):

```
1. Sunrise City             5.44   genre match (pop) (+1.5); mood match (happy) (+2.0); energy (0.82) close to your target (0.85) (+1.94)
2. Rooftop Lights           3.82   mood match (happy) (+2.0); energy (0.76) close to your target (0.85) (+1.82)
3. Gym Hero                 3.34   genre match (pop) (+1.5); energy (0.93) close to your target (0.85) (+1.84)
4. Carnival Lights          1.90   energy (0.80) close to your target (0.85) (+1.90)
5. Storm Runner             1.88   energy (0.91) close to your target (0.85) (+1.88)
```

**Chill Lofi** (`genre=lofi, mood=chill, energy=0.35, likes_acoustic=True`):

```
1. Library Rain             6.00   genre match (lofi) (+1.5); mood match (chill) (+2.0); energy (0.35) close to your target (0.35) (+2.00); matches your preference for acoustic songs (+0.5)
2. Midnight Coding          5.86   genre match (lofi) (+1.5); mood match (chill) (+2.0); energy (0.42) close to your target (0.35) (+1.86); matches your preference for acoustic songs (+0.5)
3. Spacewalk Thoughts       4.36   mood match (chill) (+2.0); energy (0.28) close to your target (0.35) (+1.86); matches your preference for acoustic songs (+0.5)
4. Focus Flow               3.90   genre match (lofi) (+1.5); energy (0.40) close to your target (0.35) (+1.90); matches your preference for acoustic songs (+0.5)
5. Coffee Shop Stories      2.46   energy (0.37) close to your target (0.35) (+1.96); matches your preference for acoustic songs (+0.5)
```

**Deep Intense Rock** (`genre=rock, mood=intense, energy=0.90, likes_acoustic=False`):

```
1. Storm Runner             5.48   genre match (rock) (+1.5); mood match (intense) (+2.0); energy (0.91) close to your target (0.90) (+1.98)
2. Gym Hero                 3.94   mood match (intense) (+2.0); energy (0.93) close to your target (0.90) (+1.94)
3. Broken Streetlights      1.90   energy (0.95) close to your target (0.90) (+1.90)
4. Sunrise City             1.84   energy (0.82) close to your target (0.90) (+1.84)
5. Carnival Lights          1.80   energy (0.80) close to your target (0.90) (+1.80)
```

**Conflicting mood/energy** (`genre=rock, mood=sad, energy=0.90, likes_acoustic=False`):

```
1. Storm Runner             3.48   genre match (rock) (+1.5); energy (0.91) close to your target (0.90) (+1.98)
2. Rainy Window Blues       2.34   mood match (sad) (+2.0); energy (0.32) close to your target (0.90) (+0.84); more acoustic than you usually like (-0.5)
3. Gym Hero                 1.94   energy (0.93) close to your target (0.90) (+1.94)
4. Broken Streetlights      1.90   energy (0.95) close to your target (0.90) (+1.90)
5. Sunrise City             1.84   energy (0.82) close to your target (0.90) (+1.84)
```

**Nonexistent genre** (`genre=k-pop, mood=happy, energy=0.80, likes_acoustic=False`):

```
1. Sunrise City             3.96   mood match (happy) (+2.0); energy (0.82) close to your target (0.80) (+1.96)
2. Rooftop Lights           3.92   mood match (happy) (+2.0); energy (0.76) close to your target (0.80) (+1.92)
3. Carnival Lights          2.00   energy (0.80) close to your target (0.80) (+2.00)
4. Night Drive Loop         1.90   energy (0.75) close to your target (0.80) (+1.90)
5. Storm Runner             1.78   energy (0.91) close to your target (0.80) (+1.78)
```

**Out-of-range energy** (`genre=pop, mood=happy, energy=1.5, likes_acoustic=False`):

```
1. Sunrise City             4.14   genre match (pop) (+1.5); mood match (happy) (+2.0); energy (0.82) close to your target (1.50) (+0.64)
2. Rooftop Lights           2.52   mood match (happy) (+2.0); energy (0.76) close to your target (1.50) (+0.52)
3. Gym Hero                 2.36   genre match (pop) (+1.5); energy (0.93) close to your target (1.50) (+0.86)
4. Broken Streetlights      0.90   energy (0.95) close to your target (1.50) (+0.90)
5. Storm Runner             0.82   energy (0.91) close to your target (1.50) (+0.82)
```

**Empty profile** (`{}`):

```
1. Sunrise City             0.00   no strong matches
2. Midnight Coding          0.00   no strong matches
3. Storm Runner             0.00   no strong matches
4. Library Rain             0.00   no strong matches
5. Gym Hero                 0.00   no strong matches
```

**Acoustic vs genre conflict** (`genre=metal, mood=angry, energy=0.95, likes_acoustic=True`):

```
1. Broken Streetlights      5.50   genre match (metal) (+1.5); mood match (angry) (+2.0); energy (0.95) close to your target (0.95) (+2.00)
2. Golden Horizon           2.00   energy (0.70) close to your target (0.95) (+1.50); matches your preference for acoustic songs (+0.5)
3. Gym Hero                 1.96   energy (0.93) close to your target (0.95) (+1.96)
4. Storm Runner             1.92   energy (0.91) close to your target (0.95) (+1.92)
5. Sunrise City             1.74   energy (0.82) close to your target (0.95) (+1.74)
```

**What surprised me:** the empty profile didn't error out, it just gave every song a tied score of 0.00 and silently fell back to CSV file order, which looks like a real recommendation but isn't one. The conflicting mood/energy profile also surprised me — I expected the one actually "sad" song to win since mood is weighted higher than genre, but a strong genre+energy combination on a non-sad song still beat it, since mood is only one of four rules, not an automatic winner.

**Does it feel right?** Taking the "Deep Intense Rock" profile (`genre=rock, mood=intense, energy=0.90`), `Storm Runner` landing at #1 feels completely correct to me — it's rock, it's labeled intense, and the energy is a near-exact match, so that one isn't even a close call. `Gym Hero` at #2 also feels right on reflection, even though its genre is pop, not rock: it's still tagged "intense" with very high energy, and a gym/pump-up pop song genuinely can scratch the same itch as intense rock for a workout mood, so I agree with the system crediting mood over genre here.

Where my intuition disagrees with the system is `Broken Streetlights` at #3. It's metal, tagged "angry," at the highest energy in the catalog (0.95) — to me, "angry metal" and "intense rock" are basically neighbors on the same emotional spectrum, closer to what I'd want than a mellow pop song. But the system only gave it credit for energy closeness (+1.90), since "angry" doesn't literally equal "intense" as a string, so it ranked behind calmer pop songs further down the list. That's a real gap between how a person hears mood (as a spectrum of similar feelings) and how the system checks mood (as an exact label match) — a fair thing to note as a limitation rather than a bug.

**Comparing profiles:**

- **High-Energy Pop vs. Chill Lofi:** these two top-5 lists don't share a single song. High-Energy Pop pulls from the fast, upbeat side of the catalog (energy around 0.85), while Chill Lofi pulls from the slow, quiet side (energy around 0.35) — basically opposite ends of the same scale. That makes sense: energy is one of the biggest things the scoring rule checks, so asking for opposite energy levels sends the two profiles to opposite ends of the catalog, exactly like asking an EDM fan and a quiet-acoustic fan for recommendations should.

- **Why does "Gym Hero" keep showing up for "Happy Pop" fans?** `Gym Hero` lands near the top of the High-Energy Pop list even though, if you actually listened to it, you'd probably call it an "intense gym song," not a "happy song." The reason is simple: the system only checks three things for that profile — is it pop (yes), is it happy (no, it's tagged "intense"), and is the energy close to what was asked for (yes, almost exactly). Two out of three is still a pretty good score, so it sneaks in near the top on genre and energy alone, even though the actual *feeling* of the song doesn't match what a "happy" listener probably wants. It's a good example of the system being able to tell loud from quiet very well, but not being able to tell happy from intense just because they're both energetic.

---

## 8. Future Work  

1. **Add a diversity rule.** Right now the top 5 can include two songs by the same artist (this happened in testing — see Section 6). I'd add a rule that skips or lowers repeat artists in the same list.

2. **Let mood be more flexible.** Right now mood has to match exactly ("intense" and "angry" are treated as totally unrelated, even though they feel similar). I'd group similar moods together so close-enough moods still earn some credit.

3. **Use real listening behavior instead of just stated preferences.** Right now the user has to type in their taste. A better version would learn from what they actually skip or replay, like real apps do (this was the collaborative filtering idea from earlier in the project).

---

## 9. Personal Reflection  

**What was my biggest learning moment?** I'm deaf and don't really listen to music, so I didn't know what things like "energy," "mood," or "tempo" even meant in a song. Using AI to explain these terms, in context of building this project, was actually how I learned them.

**How did AI tools help me, and when did I need to double-check them?** AI helped me understand music concepts I don't have personal experience with. But I still had to double-check the actual math and test the code myself, to make sure what AI explained matched what the program really did.

**What surprised me about how simple algorithms can still "feel" like recommendations?** I was surprised that a simple point-based formula could still feel like a real recommendation. But I was also surprised at how "dumb" it can be — just matching labels and numbers isn't the same as really understanding what a song feels like.

**What would I try next if I extended this project?** I'd want to spend months researching to build something closer to what Spotify or YouTube actually do — a much better algorithm, trained on a lot more real data, instead of a small list of made-up songs.
