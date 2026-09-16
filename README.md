# AFK League site

Plain HTML/CSS/JS site backed by Firestore. Five pages:

- `index.html` — home feed (live/upcoming/recent across league + tournament)
- `league.html` — AFK LEAGUE fixtures + table (up to 20 players)
- `tournament.html` — AFK TOURNAMENT bracket (8 players)
- `champions.html` — AFK ROLL OF CHAMPIONS
- `admin.html` — login-gated dashboard where the 3 admins post scores

## 1. Enable Firebase Authentication

In the [Firebase console](https://console.firebase.google.com) → your `afk-league` project:

1. **Authentication → Sign-in method** → enable **Email/Password**.
2. **Authentication → Users** → add exactly 3 users (one per admin), e.g.:
   - `admin1@afkleague.com`
   - `admin2@afkleague.com`
   - `admin3@afkleague.com`
   (use real emails/passwords for your 3 admins — these are just placeholders)
3. Open `assets/firebase.js` and update the `ADMIN_EMAILS` array to match the exact emails you created.

## 2. Firestore security rules

Go to **Firestore Database → Rules** and paste this, updating the email list to match `ADMIN_EMAILS`:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    function isAdmin() {
      return request.auth != null &&
        request.auth.token.email in [
          'admin1@afkleague.com',
          'admin2@afkleague.com',
          'admin3@afkleague.com'
        ];
    }

    match /leagueTeams/{doc}    { allow read: if true; allow write: if isAdmin(); }
    match /leagueFixtures/{doc} { allow read: if true; allow write: if isAdmin(); }
    match /tournamentPlayers/{doc} { allow read: if true; allow write: if isAdmin(); }
    match /tournamentMatches/{doc} { allow read: if true; allow write: if isAdmin(); }
    match /champions/{doc}      { allow read: if true; allow write: if isAdmin(); }
  }
}
```

This is what actually enforces "3 admins only can post" — the client-side check in `admin.html` is just a UI convenience; the Firestore rules are the real gate.

## 3. Collections created automatically

You don't need to pre-create these — the admin dashboard creates documents the first time you add something:

| Collection            | Fields |
|------------------------|--------|
| `leagueTeams`          | name, played, won, drawn, lost, gf, ga, points |
| `leagueFixtures`       | home, away, homeScore, awayScore, status (`scheduled`/`live`/`ft`), matchday, date |
| `tournamentPlayers`    | name, seed |
| `tournamentMatches`    | round, order, player1, player2, score1, score2, status, date |
| `champions`            | season, competition, winner, date |

Marking a league fixture "Full-time" for the first time automatically updates both teams' played/won/drawn/lost/GF/GA/points in the table — no manual table editing needed.

## 4. Run it

No build step. Either:

- Open `index.html` directly in a browser (works, but some browsers restrict `file://` + Firestore — a local server is safer), or
- Serve locally: `npx serve .` (or any static server) and visit `http://localhost:PORT`, or
- Deploy for free on **Firebase Hosting**:
  ```
  npm install -g firebase-tools
  firebase login
  firebase init hosting   # pick this folder as the public directory
  firebase deploy
  ```

## 5. Day-to-day use

- Public pages (`index`, `league`, `tournament`, `champions`) are read-only for everyone, no login needed.
- Only your 3 admins sign in at `admin.html` to add players/entrants, post fixtures, update scores/status, and log champions.
