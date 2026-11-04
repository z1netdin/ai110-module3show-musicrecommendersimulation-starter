# 🎵 Music Recommender Simulation

## Project Summary

In this project you will build and explain a small music recommender system.

Your goal is to:

- Represent songs and a user "taste profile" as data
- Design a scoring rule that turns that data into recommendations
- Evaluate what your system gets right and wrong
- Reflect on how this mirrors real world AI recommenders

This version, **VibeFinder 1.0**, is a content-based recommender. A user states their taste (favorite genre, favorite mood, target energy, and whether they like acoustic songs), and the system scores every song in a small catalog based on how well it matches, then returns the top 5 with plain-English reasons for each pick. It doesn't learn from other users or listening history — it only compares a song's own attributes to what the user says they want.

---

## How The System Works

Real recommenders like Spotify or YouTube mostly work in two ways: they look at what *other people* with similar taste liked (collaborative filtering), or they look at the *actual attributes* of the song or video itself, like genre and tempo, and match those to what you usually like (content-based filtering). For the collaborative side, platforms mostly watch what you *do*, not what you say: if you listen to a whole song, replay it, or click into it, that counts as a sign you liked it, and the system recommends more like it; if you skip it fast, that's a sign you didn't, so it recommends less like it. This happens automatically, from everyone's behavior, without anyone ever clicking a "like" button. Big platforms blend both approaches, but my version only does the content-based one, since I don't have a big pool of users generating listening behavior, just a small list of songs with known attributes and a one-time stated profile. My recommender takes a user's stated preferences (favorite genre, favorite mood, target energy, and whether they like acoustic songs) and compares them directly against each song's attributes, instead of learning from listening behavior over time like real systems do. I chose to prioritize mood and energy the most, since I think those two are what actually create a song's "vibe," while genre and acousticness matter too but are more about general taste than the specific feeling of a song.

Song features used: genre, mood, energy, acousticness. (tempo_bpm, valence, and danceability also exist on Song but aren't used, since UserProfile has no matching preference for them.)

UserProfile stores: favorite_genre, favorite_mood, target_energy, likes_acoustic.

To score a song, I add up points from four rules: +1.5 if the song's genre matches the user's favorite_genre, +2.0 if the mood matches favorite_mood, up to +2.0 based on how close the song's energy is to target_energy (closer energy means more points, using 2.0 * (1 - abs(song.energy - target_energy))), and +0.5/-0.5/0 depending on whether the song's acousticness lines up with likes_acoustic (bonus if it matches, penalty if it clearly conflicts, 0 if neutral). That gives each song a score out of about 6.0. I weighted mood a bit higher than genre because songs in the same genre can still feel very different (e.g. two pop songs, one happy and one intense), while songs in different genres can share the same mood, so mood felt like the stronger signal for matching vibe.

To pick which songs to recommend, I sort all the songs by that score from highest to lowest and return the top k (e.g. top 5). If two songs tie, I prefer the one with the smaller energy difference, since that's a more precise signal than the genre/mood matches.

Here's the data flow, start to finish:

```
Input: a UserProfile (favorite_genre, favorite_mood, target_energy, likes_acoustic)

Process: for every song in data/songs.csv
  - compare its genre, mood, energy, and acousticness to the UserProfile
  - add up the points from the scoring rules above
  - end up with a score (and a reason) for that one song

Output: sort every song by its score, highest first, and return the top k
```

**Algorithm Recipe (final):**

- Genre match: +1.5
- Mood match: +2.0
- Energy closeness: up to +2.0, using `2.0 * (1 - abs(song.energy - target_energy))`
- Acousticness alignment: +0.5 if it matches likes_acoustic, -0.5 if it conflicts, 0 if neutral
- Max possible score: 6.0

**Potential bias:** since I weighted mood higher than genre, this system might over-prioritize mood matches and surface songs that don't really fit a user's usual genre taste, just because the mood label happens to line up. It could also undervalue a song that's a near-perfect genre fit but only slightly off on mood. More generally, the whole system is biased toward however the genre/mood labels were assigned in the CSV, since it has no way to judge a song beyond those tags.

---

## Getting Started

### Setup

1. Create a virtual environment (optional but recommended):

   ```bash
   python -m venv .venv
   source .venv/bin/activate      # Mac or Linux
   .venv\Scripts\activate         # Windows

2. Install dependencies

```bash
pip install -r requirements.txt
```

3. Run the app:

```bash
python -m src.main
```

### Running Tests

Run the starter tests with:

```bash
pytest
```

You can add more tests in `tests/test_recommender.py`.

---

## Sample Recommendation Output

Output of `python -m src.main` using the default profile (`genre=pop, mood=happy, energy=0.8`):

```
Loaded songs: 18

Top Recommendations
----------------------------------------
1. Sunrise City by Neon Echo  (Score: 5.46)
     - genre match (pop) (+1.5)
     - mood match (happy) (+2.0)
     - energy (0.82) close to your target (0.80) (+1.96)

2. Rooftop Lights by Indigo Parade  (Score: 3.92)
     - mood match (happy) (+2.0)
     - energy (0.76) close to your target (0.80) (+1.92)

3. Gym Hero by Max Pulse  (Score: 3.24)
     - genre match (pop) (+1.5)
     - energy (0.93) close to your target (0.80) (+1.74)

4. Carnival Lights by Luna Ray  (Score: 2.00)
     - energy (0.80) close to your target (0.80) (+2.00)

5. Night Drive Loop by Neon Echo  (Score: 1.90)
     - energy (0.75) close to your target (0.80) (+1.90)
```

**Screenshot or video** *(optional)*: <!-- Insert a screenshot or demo video link here -->

---

## Experiments You Tried

Use this section to document the experiments you ran. For example:

- What happened when you changed the weight on genre from 2.0 to 0.5
- What happened when you added tempo or valence to the score
- How did your system behave for different types of users

---

## Limitations and Risks

Summarize some limitations of your recommender.

Examples:

- It only works on a tiny catalog
- It does not understand lyrics or language
- It might over favor one genre or mood

You will go deeper on this in your model card.

---

## Reflection

Read and complete `model_card.md`:

[**Model Card**](model_card.md)

Write 1 to 2 paragraphs here about what you learned:

- about how recommenders turn data into predictions
- about where bias or unfairness could show up in systems like this



