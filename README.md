# node-js-project
API'S to perform operations of sql table.

const express = require("express");
const path = require("path");

const { open } = require("sqlite");
const sqlite3 = require("sqlite3");
const app = express();
app.use(express.json());
const dbPath = path.join(__dirname, "cricketTeam.db");

let db = null;

const initializeDBAndServer = async () => {
  try {
    db = await open({
      filename: dbPath,
      driver: sqlite3.Database,
    });
    app.listen(3000, () => {
      console.log("Server Running at http://localhost:3000/");
    });
  } catch (e) {
    console.log(`DB Error: ${e.message}`);
    process.exit(1);
  }
};

initializeDBAndServer();

const convertDbObjectToResponseObject = (dbObject) => {
  return {
    playerId: dbObject.player_id,
    playerName: dbObject.player_name,
    jerseyNumber: dbObject.jersey_number,
    role: dbObject.role,
  };
};

app.get("/players/", async (request, response) => {
  const getTeamQuery = `
    SELECT
      *
    FROM
      cricket_team`;
  const playersArray = await db.all(getTeamQuery);
  response.send(
    playersArray.map((eachPlayer) =>
      convertDbObjectToResponseObject(eachPlayer)
    )
  );
});

app.post("/players/", async (request, response) => {
  const { playerName, jerseyNumber, role } = request.body;
  const postPlayerQuery = `
    INSERT INTO 
    cricket_team (player_name, jersey_number, role)
    VALUES
     ('${playerName}',${jerseyNumber},'${role}');`;
  const player = await db.run(postPlayerQuery);
  response.send("Player Added to Team");
});

app.get("/players/:playerId/", async (request, response) => {
  const { playerId } = request.params;
  const playerQuery = `
    SELECT
     *
    FROM
     cricket_team
    WHERE
     player_id=${playerId};`;
  const player = await db.get(playerQuery);
  response.send(convertDbObjectToResponseObject(player));
});

app.put("/players/:playerId/", async (request, response) => {
  const { playerName, jerseyNumber, role } = request.body;
  const { playerId } = request.params;
  const updateQuery = `
    UPDATE
    cricket_team
    SET
    player_name='${playerName}',
    jersey_number=${jerseyNumber},
    role='${role}'
    WHERE
     player_id=${playerId}`;

  await db.run(updateQuery);
  response.send("Player Details Updated");
});

app.delete("/players/:playerId/", async (request, response) => {
  const { playerId } = request.params;
  const deletePlayer = `
    DELETE FROM 
    cricket_team
    WHERE
    player_id= ${playerId};`;
  await db.run(deletePlayer);
  response.send("Player Removed");
});
module.exports = app;
