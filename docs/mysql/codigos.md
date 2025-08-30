-- ¿Cuáles fueron las 10 mejores películas según la puntuación de IMDB?
SELECT title, 
type, 
imdb_score
FROM shows_movies.titles
WHERE imdb_score >= 8.0
AND type = 'MOVIE'
ORDER BY imdb_score DESC
LIMIT 10;

-- ¿Cuáles fueron los 10 mejores programas según la puntuación de IMDB? 
SELECT title, 
type, 
imdb_score
FROM shows_movies.titles
WHERE imdb_score >= 8.0
AND type = 'SHOW'
ORDER BY imdb_score DESC
LIMIT 10;

-- ¿Cuáles fueron las 10 peores películas según la puntuación de IMDB? 
SELECT title, 
type, 
imdb_score
FROM shows_movies.titles
WHERE type = 'MOVIE'
ORDER BY imdb_score ASC
LIMIT 10;

-- ¿Cuáles fueron los 10 peores programas según la puntuación de IMDB? 
SELECT title, 
type, 
imdb_score
FROM shows_movies.titles
WHERE type = 'SHOW'
ORDER BY imdb_score ASC
LIMIT 10;

-- ¿Cuáles fueron las puntuaciones promedio de IMDB y TMDB para programas y películas? 
SELECT DISTINCT type, 
ROUND(AVG(imdb_score),2) AS avg_imdb_score,
ROUND(AVG(tmdb_score),2) as avg_tmdb_score
FROM shows_movies.titles
GROUP BY type; 

-- Recuento de películas y programas en cada década
SELECT CONCAT(FLOOR(release_year / 10) * 10, 's') AS decade,
    COUNT(*) AS movies_shows_count
FROM shows_movies.titles
WHERE release_year >= 1940
GROUP BY CONCAT(FLOOR(release_year / 10) * 10, 's')
ORDER BY decade;

-- ¿Cuáles fueron las puntuaciones promedio de IMDB y TMDB para cada país de producción?
SELECT DISTINCT production_countries, 
ROUND(AVG(imdb_score),2) AS avg_imdb_score,
ROUND(AVG(tmdb_score),2) AS avg_tmdb_score
FROM shows_movies.titles
GROUP BY production_countries
ORDER BY avg_imdb_score DESC;

-- ¿Cuáles fueron las puntuaciones promedio de IMDB y TMDB para cada clasificación de edad en programas y películas?
SELECT DISTINCT age_certification, 
ROUND(AVG(imdb_score),2) AS avg_imdb_score,
ROUND(AVG(tmdb_score),2) AS avg_tmdb_score
FROM shows_movies.titles
GROUP BY age_certification
ORDER BY avg_imdb_score DESC;

-- ¿Cuáles fueron las 5 clasificaciones de edad más comunes para las películas?
SELECT age_certification, 
COUNT(*) AS certification_count
FROM shows_movies.titles
WHERE type = 'Movie' 
AND age_certification != 'N/A'
GROUP BY age_certification
ORDER BY certification_count DESC
LIMIT 5;

-- ¿Quiénes fueron los 20 actores que más aparecieron en películas/programas? 
SELECT DISTINCT name as actor, 
COUNT(*) AS number_of_appearences 
FROM shows_movies.credits
WHERE role = 'actor'
GROUP BY name
ORDER BY number_of_appearences DESC
LIMIT 20;

-- ¿Quiénes fueron los 20 directores que más películas/programas dirigieron? 
SELECT DISTINCT name as director, 
COUNT(*) AS number_of_appearences 
FROM shows_movies.credits
WHERE role = 'director'
GROUP BY name
ORDER BY number_of_appearences DESC
LIMIT 20;

-- Calculando el tiempo de ejecución promedio de películas y programas de televisión por separado
SELECT 
'Movies' AS content_type,
ROUND(AVG(runtime),2) AS avg_runtime_min
FROM shows_movies.titles
WHERE type = 'Movie'
UNION ALL
SELECT 
'Show' AS content_type,
ROUND(AVG(runtime),2) AS avg_runtime_min
FROM shows_movies.titles
WHERE type = 'Show';

-- Encontrando los títulos y directores de las películas estrenadas en o después de 2010
SELECT DISTINCT t.title, c.name AS director, 
release_year
FROM shows_movies.titles AS t
JOIN shows_movies.credits AS c 
ON t.id = c.id
WHERE t.type = 'Movie' 
AND t.release_year >= 2010 
AND c.role = 'director'
ORDER BY release_year DESC;

-- ¿Qué programas de Netflix tienen más temporadas?
SELECT title, 
SUM(seasons) AS total_seasons
FROM shows_movies.titles 
WHERE type = 'Show'
GROUP BY title
ORDER BY total_seasons DESC
LIMIT 10;

-- ¿Qué géneros tuvieron más películas? 
SELECT genres, 
COUNT(*) AS title_count
FROM shows_movies.titles 
WHERE type = 'Movie'
GROUP BY genres
ORDER BY title_count DESC
LIMIT 10;

-- ¿Qué géneros tuvieron más programas? 
SELECT genres, 
COUNT(*) AS title_count
FROM shows_movies.titles 
WHERE type = 'Show'
GROUP BY genres
ORDER BY title_count DESC
LIMIT 10;

-- Títulos y directores de películas con altas puntuaciones de IMDB (>7.5) y altas puntuaciones de popularidad de TMDB (>80) 
SELECT t.title, 
c.name AS director
FROM shows_movies.titles AS t
JOIN shows_movies.credits AS c 
ON t.id = c.id
WHERE t.type = 'Movie' 
AND t.imdb_score > 7.5 
AND t.tmdb_popularity > 80 
AND c.role = 'director';

-- ¿Cuál fue el número total de títulos por año? 
SELECT release_year, 
COUNT(*) AS title_count
FROM shows_movies.titles 
GROUP BY release_year
ORDER BY release_year DESC;

-- Actores que han protagonizado en los títulos de películas o programas más altamente calificados
SELECT c.name AS actor, 
COUNT(*) AS num_highly_rated_titles
FROM shows_movies.credits AS c
JOIN shows_movies.titles AS t 
ON c.id = t.id
WHERE c.role = 'actor'
AND (t.type = 'Movie' OR t.type = 'Show')
AND t.imdb_score > 8.0
AND t.tmdb_score > 8.0
GROUP BY c.name
ORDER BY num_highly_rated_titles DESC;

-- ¿Qué actores/actrices interpretaron el mismo personaje en múltiples películas o programas de televisión? 
SELECT c.name AS actor_actress, 
c.character, 
COUNT(DISTINCT t.title) AS num_titles
FROM shows_movies.credits AS c
JOIN shows_movies.titles AS t 
ON c.id = t.id
WHERE c.role = 'actor' OR c.role = 'actress'
GROUP BY c.name, c.character
HAVING COUNT(DISTINCT t.title) > 1;

-- ¿Cuáles fueron los 3 géneros más comunes?
SELECT t.genres, 
COUNT(*) AS genre_count
FROM shows_movies.titles AS t
WHERE t.type = 'Movie'
GROUP BY t.genres
ORDER BY genre_count DESC
LIMIT 3;

-- Puntuación promedio de IMDB para actores/actrices principales en películas o programas 
SELECT c.name AS actor_actress, 
ROUND(AVG(t.imdb_score),2) AS average_imdb_score
FROM shows_movies.credits AS c
JOIN shows_movies.titles AS t 
ON c.id = t.id
WHERE c.role = 'actor' OR c.role = 'actress'
AND c.character = 'leading role'
GROUP BY c.name
ORDER BY average_imdb_score DESC;
