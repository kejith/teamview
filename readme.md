# 0game Teamview

## Other Scripts

- `setupDb.js`: run once to initialize local postgresdb, credentials provided via `.env`
- `importStats.js`: loads the /stats.json file from the server and dumps it into the players table (does not update, creates new entry each time)

## Required use cases

| usecase | auth level | payload (H:eader) | route | returns |
|---------|------------|------------|-------|---------|
| create API token (=new user) | none | H:password, username/comment | POST /`v1/tokens` | token |
| delete API token | password | H:password | DELETE `/v1/tokens/<token>`| success |
| create a team | token | token, teamName | POST /v1/teams | team ID, teamCode (required to join) |
| retrieve a teams passcode | none (you must be part of the team) | none | GET `/v1/teams/<teamID>/passcode` | passcode
| join a team | password | H:password, H:teamCode, token | POST `/v1/teams/join` | success |
| leave a team | password | H:password, token | DELETE `/v1/teams/<teamID>/members` | success |
| add planet data | none | array of planet info, token | POST `/v1/planets`
| add player data | none | array of player info, token | POST `/v1/players`

Limitations:
1. deleting an API token or leaving a team does not remove the data that has been already sent
1. there is no way to delete submitted data
1. an API tokens password cannot be recovered
1. deleting a team is only possible for the last remaining member

## DB structure

```mermaid
classDiagram
  teamMembers --> teams
  teamMembers --> tokens
  planets --> players
  planets --> teams
  research --> players
  class tokens {
    id: int
    name: string
    value: string
    password: string [hashed]
  }
  class teams {
    id: int
    name: string
    code: string
  }
  class teamMembers {
    id: int
    tokenId: int (tokens.id)
    teamId: int (teams.id)
  }
  class planets {
    id: int
    universe: int
    galaxy: int
    system: int
    position: int
    resources: json
    buildings: json
    fleet: json
    defense: json
    gamePlanetId: int
    playerId: int (players.id)
    access: int (teams.id)
  }
  class players {
    id: int
    name: string
    alliance: string
    rank: int
    pointsResearch: int
    pointsDefense: int
    pointsFleet: int
    unitsDestroyed: int
    unitsLost: int
    battlesLost: int
    battlesWon: int
    battlesDraw: int
    research: json
    access: int (teams.id)
  }
  class research {
    id: int
    stuff...
    playerId: int (players.id)
    access: int (teams.id)
  }
```

## Ideas

### Prevent API spam by implementing rate limit

Especially for POST/DELETE operations

- either via proxy https://faun.pub/prevent-ddos-attacks-with-traefik-2-44fb32eeac4f
- or in app https://www.npmjs.com/package/express-slow-down
