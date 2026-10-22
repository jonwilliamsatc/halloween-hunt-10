# Halloween Hunt 10 — 8-riddle image-clue app

This is a responsive haunted-mansion web app with:
- Eight riddles playable in **any order**
- Typed answers, case/spacing/punctuation-insensitive matching
- Incorrect answers show a retry message
- Each image clue is returned only after its riddle is correctly answered
- Solved clues remain unlocked and can be viewed again
- Progress saved in Supabase and resumable on the same browser
- Shared leaderboard of completed hunts ranked by elapsed time
- Free hosting options: GitHub Pages for the frontend, Supabase free tier for backend/database/storage (subject to current plan limits)

## Files
- `index.html` — frontend
- `supabase/schema.sql` — database table
- `supabase/functions/hunt/index.ts` — server-side answer checking and signed private image links
- `clues/README.txt` — clue image naming instructions

## 1. Create Supabase project
1. Create a project at https://supabase.com.
2. In **Project Settings → API**, copy the Project URL and public `anon` / publishable key.
3. In **SQL Editor**, run all of `supabase/schema.sql`.
4. Open **Storage** and create a bucket named `clue-images`. Set it to **Private** (not public).

## 2. Upload your eight clue images
Upload your images into the private `clue-images` bucket with these exact object names:
- `clue-01.jpg`
- `clue-02.jpg`
- `clue-03.jpg`
- `clue-04.jpg`
- `clue-05.jpg`
- `clue-06.jpg`
- `clue-07.jpg`
- `clue-08.jpg`

JPG, PNG, and WebP are fine, but if you use another extension, update the matching `image_path` in `index.ts`. These are the actual clue images shown to players, so make each image clearly point to its next hiding place. Keep the bucket private: the server issues a time-limited signed URL only after the player has unlocked that clue.

## 3. Customize the riddles and answers
Edit `supabase/functions/hunt/index.ts`. Each object in `riddles` includes:
- `title`, `icon`, `prompt`
- `answers`: accepted typed answers (include alternative spellings)
- `image_path`: filename of the clue image in private storage
- `caption`: short text displayed with the image

Replace the example riddles and accepted answers to fit your event. The eight example riddle answers are `tombstone`, `mirror`, `black cat`, `jack-o-lantern`, `key`, `cauldron`, `ghost`, and `treasure chest`. Adjust as needed. Answer matching ignores capitalization, punctuation, and repeated spaces.

## 4. Deploy the backend function
Install the Supabase CLI per https://supabase.com/docs/guides/cli, then in this project folder run:

```bash
supabase login
supabase link --project-ref YOUR_PROJECT_REF
supabase functions deploy hunt
```

The Edge Function uses Supabase's server-side `SUPABASE_URL` and `SUPABASE_SERVICE_ROLE_KEY`. Never put the service-role key into `index.html`. If your project does not automatically expose these to the function, set them in the Edge Function secrets dashboard.

## 5. Configure the frontend
Open `index.html`, near the bottom, and replace:

```js
const SUPABASE_URL = "https://cexxbycchktjojowfsfg.supabase.co/rest/v1/functions/v1/hunt";
const SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImNleHhieWNjaGt0am9qb3dmc2ZnIiwicm9sZSI6ImFub24iLCJpYXQiOjE3OTI2NjI5OTQsImV4cCI6MjEwODIzODk5NH0.yBP4nXhS8jradoU_9qeaVivZxOzddvW9V6gEe7kwCdM";
```

with your project URL and public anon/publishable key. The anon key is designed to be public; never use a service-role key here.

## 6. Publish free with GitHub Pages
1. Create a GitHub repository (e.g. `haunted-hunt`).
2. Upload `index.html` to the repository root. You do not need to upload the private clue images to GitHub; they live in Supabase Storage.
3. Go to repository **Settings → Pages**.
4. Choose **Deploy from a branch**, select `main` and `/ (root)`, then save.
5. Wait for GitHub Pages to publish and open the URL displayed there.
6. Test on a phone and another browser before the event.

## How the app behaves
- The player chooses any of the eight cases from the grid.
- The answer is checked on the server. Incorrect answers do not unlock anything.
- On a correct answer, the backend saves the solved case and returns a signed URL for its private image clue.
- The solved case becomes unlocked in the grid; players can revisit it.
- Refreshing or returning on the same browser resumes using the stored session ID and server-side progress.
- After all eight are solved, the app records the completion time and adds the player to the leaderboard.

## Testing checklist
- Try an incorrect answer: no clue should appear.
- Solve a case: its image appears and its tile changes to “Clue unlocked”.
- Refresh: solved tiles and images should restore.
- Solve cases in a different order: all should work.
- Finish all eight: leaderboard should show the completed run.
- Check that the storage bucket is private and the answer key exists only in the Edge Function.

## Practical limitations and safety
This is intended for a casual party. The session ID is stored in that browser; it is not an account system and won't automatically transfer to another device. Anyone who has a session ID could potentially resume that run, so do not share it. Public leaderboard names are visible to participants; nicknames are recommended. The public function can be spammed, so for a large public event, add rate limiting and stronger identity/recovery controls.
