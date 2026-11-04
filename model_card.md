# 🎧 Model Card: Music Recommender Simulation

## 1. Model Name  

Give your model a short, descriptive name.  
Example: **VibeFinder 1.0**  

---

## 2. Intended Use  

Describe what your recommender is designed to do and who it is for. 

Prompts:  

- What kind of recommendations does it generate  
- What assumptions does it make about the user  
- Is this for real users or classroom exploration  

---

## 3. How the Model Works  

This recommender doesn't learn from what anyone else listened to. It just compares each song's own attributes to what the user says they want, and scores how well they line up. Four things matter: the song's genre, its mood, how energetic it is, and how acoustic it sounds. The user's taste profile says which genre and mood they like, roughly how much energy they want, and whether they tend to like acoustic songs.

To get a score, the song earns points for each thing that lines up: a bit of credit for matching genre, a bit more for matching mood (I decided mood should count for more, since two songs in the same genre can still feel completely different, while songs in different genres can share the same vibe), and a sliding amount of credit for energy depending on how close it is to what the user wants, not just whether it's high or low. There's also a small bonus or penalty depending on whether the song's acoustic-ness fits what the user usually likes. All of that gets added up into one number, and the song also gets a plain-English list of reasons for why it scored the way it did, so a user isn't just shown a number out of nowhere.

Once every song in the catalog has a score, the system just sorts them from highest to lowest and hands back the top handful. The starter code originally just returned the first few songs in the file with no real scoring at all, so the main change is that recommendations are now actually based on matching the user's stated preferences instead of file order.

---

## 4. Data  

Describe the dataset the model uses.  

Prompts:  

- How many songs are in the catalog  
- What genres or moods are represented  
- Did you add or remove data  
- Are there parts of musical taste missing in the dataset  

---

## 5. Strengths  

Where does your system seem to work well  

Prompts:  

- User types for which it gives reasonable results  
- Any patterns you think your scoring captures correctly  
- Cases where the recommendations matched your intuition  

---

## 6. Limitations and Bias 

Where the system struggles or behaves unfairly. 

Prompts:  

- Features it does not consider  
- Genres or moods that are underrepresented  
- Cases where the system overfits to one preference  
- Ways the scoring might unintentionally favor some users  

---

## 7. Evaluation  

How you checked whether the recommender behaved as expected. 

Prompts:  

- Which user profiles you tested  
- What you looked for in the recommendations  
- What surprised you  
- Any simple tests or comparisons you ran  

No need for numeric metrics unless you created some.

---

## 8. Future Work  

Ideas for how you would improve the model next.  

Prompts:  

- Additional features or preferences  
- Better ways to explain recommendations  
- Improving diversity among the top results  
- Handling more complex user tastes  

---

## 9. Personal Reflection  

A few sentences about your experience.  

Prompts:  

- What you learned about recommender systems  
- Something unexpected or interesting you discovered  
- How this changed the way you think about music recommendation apps  
