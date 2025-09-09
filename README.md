# Internship-Project

 Sports Tournament Tracker
 
 Objective: Manage match results and player statistics.

 -------------------------------------------------------------
-- STEP 1: CREATE DATABASE
-------------------------------------------------------------
CREATE DATABASE SportsTournament;
USE SportsTournament;

-------------------------------------------------------------
-- STEP 2: CREATE TABLES
-------------------------------------------------------------

-- 1. Teams Table
CREATE TABLE Teams (
    team_id INT PRIMARY KEY IDENTITY(1,1),
    team_name VARCHAR(20) NOT NULL,
    city VARCHAR(20)
);
GO

SELECT * FROM Teams

-- 2. Players Table
CREATE TABLE Players (
    player_id INT PRIMARY KEY IDENTITY(1,1),
    player_name VARCHAR(20) NOT NULL,
    team_id INT,
    role VARCHAR(20),
    CONSTRAINT FK_Players_Teams FOREIGN KEY (team_id)
    REFERENCES Teams(team_id)
);

GO

SELECT * FROM Players


-- 3. Matches Table
CREATE TABLE Matches (
    match_id INT PRIMARY KEY IDENTITY(1,1),
    match_date DATE,
    team1_id INT FOREIGN KEY (team1_id) REFERENCES Teams(team_id),
    team2_id INT FOREIGN KEY (team2_id) REFERENCES Teams(team_id),
    team1_score INT,
    team2_score INT,
    winner_team_id INT FOREIGN KEY (winner_team_id) REFERENCES Teams(team_id)
);
GO
SELECT * FROM Matches

-- 4. PlayerStats Table
CREATE TABLE PlayerStats (
    stat_id INT PRIMARY KEY IDENTITY(1,1),
    match_id INT FOREIGN KEY (match_id) REFERENCES Matches(match_id),
    player_id INT FOREIGN KEY (player_id) REFERENCES Players(player_id),
    runs INT DEFAULT 0,
    wickets INT DEFAULT 0,
);
GO
SELECT * FROM PlayerStats

-------------------------------------------------------------
-- STEP 3: INSERT SAMPLE DATA
-------------------------------------------------------------

-- Insert Teams
INSERT INTO Teams (team_name, city) VALUES
('Warriors', 'Mumbai'),
('Titans', 'Delhi'),
('Riders', 'Bangalore'),
('Kings', 'Hyderabad'),
('Giants','Chennai');
GO
SELECT * FROM Teams

-- Insert Players
INSERT INTO Players (player_name, team_id, role) VALUES
('Rohit Sharma', 1, 'Batsman'),
('Jasprit Bumrah', 1, 'Bowler'),
('Virat Kohli', 3, 'Batsman'),
('AB de Villiers', 3, 'Batsman'),
('David Warner', 4, 'Batsman'),
('Rashid Khan', 4, 'Bowler'),
('Shubman Gill', 2, 'Batsman'),
('Mohammed Shami', 2, 'Bowler'),
('M S Dhoni',5,'WK Batsman'),
('Ravindra Jadeja',5,'ALL Rounder');
GO
SELECT * FROM Players


-- Insert Matches
INSERT INTO Matches (match_date, team1_id, team2_id, team1_score, team2_score, winner_team_id) VALUES
('2025-08-01', 1, 2, 90, 77, 1),
('2025-08-02', 3, 4, 127, 73, 3),
('2025-08-03', 5, 2, 97, 84, 5),
('2025-08-04', 1, 3, 64, 66, 3),
('2025-08-05', 2, 4, 135, 120, 2),
('2025-08-06', 1, 5, 120, 103, 1),
('2025-08-07', 1, 4, 107, 109, 4),
('2025-08-08', 2, 3, 117, 120, 3),
('2025-08-09', 4, 5, 105, 106, 5),
('2025-08-10', 5, 3, 84, 67, 5);
GO
SELECT * FROM Matches


-- Insert Player Stats
INSERT INTO PlayerStats (match_id, player_id, runs, wickets) VALUES
(1, 1, 70, 0),
(1, 2, 15, 3),
(1, 7, 65, 0),
(1, 8, 10, 2),

(2, 3, 80, 0),
(2, 4, 45, 0),
(2, 5, 60, 0),
(2, 6, 10, 4),

(3, 9, 73, 0),
(3, 10, 49, 2),
(3, 7, 68, 0),
(3, 8, 20, 0),

(4, 1, 48, 0),
(4, 2, 14, 3),
(4, 3, 40, 0),
(4, 4, 25, 2),

(5, 7, 110, 0),
(5, 8, 24, 3),
(5, 5, 98, 0),
(5, 6, 20, 2),

(6, 1, 88, 0),
(6, 2, 37, 3),
(6, 9, 64, 0),
(6, 10, 38, 2),

(7, 1, 100, 0),
(7, 2, 5, 3),
(7, 5, 90, 0),
(7, 6, 16, 2),

(8, 7, 102, 0),
(8, 8, 12, 3),
(8, 3, 80, 0),
(8, 4, 37, 2),

(9, 5, 70, 0),
(9, 6, 32, 3),
(9, 9, 85, 0),
(9, 10, 20, 2),

(10, 9, 20, 0),
(10, 10, 62, 3),
(10, 3, 40, 0),
(10, 4, 25, 2);
GO
SELECT * FROM PlayerStats


-------------------------------------------------------------
-- STEP 4: BASIC QUERIES
-------------------------------------------------------------

-- Show all match results
SELECT m.match_id, m.match_date,
       t1.team_name AS Team1,
       t2.team_name AS Team2,
       m.team1_score, m.team2_score,
       t3.team_name AS Winner
FROM Matches m
JOIN Teams t1 ON m.team1_id = t1.team_id
JOIN Teams t2 ON m.team2_id = t2.team_id
JOIN Teams t3 ON m.winner_team_id = t3.team_id
ORDER BY m.match_date;
GO

-- Top 5 Run Scorers
SELECT TOP 5 p.player_name, SUM(ps.runs) AS total_runs
FROM PlayerStats ps
JOIN Players p ON ps.player_id = p.player_id
GROUP BY p.player_name
ORDER BY total_runs DESC;
GO

-- Top 5 Wicket Takers
SELECT TOP 5 p.player_name, SUM(ps.wickets) AS total_wickets
FROM PlayerStats ps
JOIN Players p ON ps.player_id = p.player_id
GROUP BY p.player_name
ORDER BY total_wickets DESC;
GO

-------------------------------------------------------------
-- STEP 5: CREATE LEADERBOARD VIEW
-------------------------------------------------------------
CREATE VIEW TeamLeaderboard AS
SELECT t.team_name,
       COUNT(CASE WHEN m.winner_team_id = t.team_id THEN 1 END) AS matches_won,
       COUNT(m.match_id) AS matches_played,
       COUNT(CASE WHEN m.winner_team_id = t.team_id THEN 1 END) * 2 AS points
FROM Teams t
LEFT JOIN Matches m
ON t.team_id IN (m.team1_id, m.team2_id)
GROUP BY t.team_name;
GO

-- View Leaderboard
SELECT * FROM TeamLeaderboard;
GO

-------------------------------------------------------------
-- STEP 6: AVERAGE PLAYER PERFORMANCE USING CTE
-------------------------------------------------------------
WITH PlayerPerformance AS (
    SELECT p.player_name,
           SUM(ps.runs) AS total_runs,
           SUM(ps.wickets) AS total_wickets,
           COUNT(DISTINCT ps.match_id) AS matches_played
    FROM PlayerStats ps
    JOIN Players p ON ps.player_id = p.player_id
    GROUP BY p.player_name
)
SELECT player_name,
       CAST(total_runs * 1.0 / matches_played AS DECIMAL(10,2)) AS avg_runs,
       CAST(total_wickets * 1.0 / matches_played AS DECIMAL(10,2)) AS avg_wickets
FROM PlayerPerformance
ORDER BY avg_runs DESC;
GO


-- ------------------------------------------
-- STEP 7: EXPORT TEAM PERFORMANCE REPORT
-- ------------------------------------------

SELECT * FROM TeamLeaderboard;

