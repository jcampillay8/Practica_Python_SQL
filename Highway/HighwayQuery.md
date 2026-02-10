```
-- ==========================================
-- ESTRUCTURA DE LA BASE DE DATOS: HIGHWAY TO DATA HERO
-- ==========================================

-- 1. Catálogo Maestro de Bandas
CREATE TABLE bands_artists (
    artist_id SERIAL PRIMARY KEY,
    artist_name VARCHAR(100) NOT NULL,
    origin_country VARCHAR(50),
    genre VARCHAR(30),
    year_formed INT
);

-- 2. Audiencia (Fans)
CREATE TABLE music_fans (
    fan_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50),
    country VARCHAR(50),
    local_currency CHAR(3) NOT NULL,
    wallet_balance DECIMAL(15, 2) DEFAULT 0.00,
    member_level VARCHAR(10) CHECK (member_level IN ('Gold', 'Silver', 'Bronze'))
);

-- 3. Planificación de Tours
CREATE TABLE concert_tours (
    tour_id SERIAL PRIMARY KEY,
    artist_id INT REFERENCES bands_artists(artist_id) ON DELETE CASCADE,
    venue_name VARCHAR(100),
    city VARCHAR(50),
    country VARCHAR(50),
    tour_date DATE NOT NULL,
    start_time TIME
);

-- 4. Lógica de Precios (Multidivisa)
CREATE TABLE ticket_prices (
    price_id SERIAL PRIMARY KEY,
    tour_id INT REFERENCES concert_tours(tour_id) ON DELETE CASCADE,
    tier_name VARCHAR(20) CHECK (tier_name IN ('Premium', 'Cancha', 'Galería')),
    base_price DECIMAL(15, 2) NOT NULL,
    event_currency CHAR(3) NOT NULL
);

-- 5. Motor de Conversión de Divisas
-- Nota: La PK es compuesta para asegurar que solo haya una tasa por par de monedas
CREATE TABLE exchange_rates (
    from_currency CHAR(3),
    to_currency CHAR(3),
    conversion_factor DECIMAL(18, 6) NOT NULL,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (from_currency, to_currency)
);

-- 6. Jerarquía del Staff (Soporta Self-Joins)
CREATE TABLE staff_hierarchy (
    staff_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    role VARCHAR(50),
    salary DECIMAL(12, 2),
    manager_id INT REFERENCES staff_hierarchy(staff_id), -- Self-Join
    tour_id INT REFERENCES concert_tours(tour_id)
);

-- 7. Histórico de Tours (Para UNION / UNION ALL)
CREATE TABLE tour_history_archive (
    archive_id SERIAL PRIMARY KEY,
    artist_id INT, -- Aquí no ponemos FK estricta para permitir datos de bandas que quizás ya no existen
    venue_name VARCHAR(100),
    city VARCHAR(50),
    country VARCHAR(50),
    tour_date DATE,
    start_time TIME
);


INSERT INTO bands_artists (artist_name, origin_country, genre, year_formed) VALUES
(' AC/DC ', 'Australia', 'Hard Rock', 1973),                    -- Espacios para TRIM
('Metallica', 'USA', 'Thrash Metal', 1981),
('Led Zeppelinn', 'UK', 'Hard Rock', 1968),                      -- Error de tipeo
('Queen', 'UK', 'Rock', 1970),
('Black Sabbath', 'UK', 'Heavy Metal', 1968),
('iron maiden', 'UK', 'NWOBHM', 1975),                           -- Minúsculas para INITCAP
('Guns N'' Roses', 'USA', 'Hard Rock', 1985),                   -- Escape de comilla
('Deep Purple', 'UK', 'Hard Rock', 1968),                       -- En lugar de Stones
('Nirvana', 'USA', 'Grunge', 1987),                             
('Pink Floyd', 'UK', 'Progressive Rock', 1965);

INSERT INTO music_fans (first_name, last_name, country, local_currency, wallet_balance, member_level) VALUES
('John', 'Smith', 'USA', 'USD', 150.00, 'Gold'),
('Hans', 'Müller', 'Germany', 'EUR', 120.00, 'Silver'),
('Diego', 'Soto', 'Chile', 'CLP', 85000.00, 'Bronze'),           -- Saldo alto en número, bajo en valor real
('Thiago', 'Silva', 'Brazil', 'BRL', 450.00, 'Gold'),
(' Mick ', 'Jagger', 'UK', 'GBP', 5000.00, 'Gold'),               -- Espacios en nombre para TRIM
('Sarah', NULL, 'Canada', 'CAD', 200.00, 'Silver'),               -- NULL en apellido para COALESCE
('Alice', 'Cooper', 'USA', 'USD', 300.00, 'Gold'),
('Pedro', 'Gomez', 'Argentina', 'ARS', 150000.00, 'Bronze'),      -- Caso de inflación/números grandes
('Maria', 'Garcia', 'Mexico', 'MXN', 2500.00, 'Silver'),
('Bob', NULL, 'USA', 'USD', 45.00, 'Bronze');                     -- Saldo insuficiente para casi todo

INSERT INTO concert_tours (artist_id, venue_name, city, country, tour_date, start_time) VALUES
(1, 'Estadio Nacional', 'Santiago', 'Chile', '2024-11-15', '21:00:00'),     -- AC/DC (Para el fan chileno)
(2, 'Wembley Stadium', 'London', 'UK', '2024-06-20', '20:00:00'),            -- Metallica (Para el fan inglés)
(2, 'River Plate', 'Buenos Aires', 'Argentina', '2024-12-01', '21:30:00'),   -- Metallica (Para el fan argentino)
(3, 'Madison Square Garden', 'New York', 'USA', '1973-07-27', '20:00:00'),   -- Led Zeppelin (Histórico)
(4, 'Live Aid - Wembley', 'London', 'UK', '1985-07-13', '18:41:00'),         -- Queen (Momento exacto)
(5, 'O2 Arena', 'London', 'UK', '2025-02-10', '19:30:00'),                   -- Black Sabbath
(6, 'The Forum', 'Los Angeles', 'USA', '2024-10-05', '20:00:00'),            -- Iron Maiden
(7, 'Dodger Stadium', 'Los Angeles', 'USA', '1992-10-03', '21:00:00'),       -- Guns N' Roses
(9, 'Reading Festival', 'Reading', 'UK', '1992-08-30', '22:00:00'),          -- Nirvana
(10, 'Pompeii Amphitheatre', 'Pompeii', 'Italy', '1971-10-04', '12:00:00');  -- Pink Floyd

INSERT INTO ticket_prices (tour_id, tier_name, base_price, event_currency) VALUES
-- AC/DC en Chile (tour_id 1) - Moneda: CLP
(1, 'Premium', 150000.00, 'CLP'),
(1, 'Cancha', 85000.00, 'CLP'),
(1, 'Galería', 45000.00, 'CLP'),

-- Metallica en UK (tour_id 2) - Moneda: GBP
(2, 'Premium', 250.00, 'GBP'),
(2, 'Cancha', 120.00, 'GBP'),
(2, 'Galería', 75.00, 'GBP'),

-- Metallica en Argentina (tour_id 3) - Moneda: ARS
(3, 'Premium', 95000.00, 'ARS'),
(3, 'Cancha', 45000.00, 'ARS'),
(3, 'Galería', 25000.00, 'ARS'),

-- Iron Maiden en USA (tour_id 7) - Moneda: USD
(7, 'Premium', 350.00, 'USD'),
(7, 'Cancha', 150.00, 'USD'),
(7, 'Galería', 85.00, 'USD'),

-- Queen en UK (tour_id 5 - Histórico/Live Aid) - Moneda: GBP
(5, 'Premium', 50.00, 'GBP'), -- Precios de 1985
(5, 'Cancha', 25.00, 'GBP'),

-- Pink Floyd en Italia (tour_id 10) - Moneda: EUR
(10, 'Premium', 100.00, 'EUR');

INSERT INTO exchange_rates (from_currency, to_currency, conversion_factor) VALUES
-- De Monedas Locales a Dólar (Para estandarizar)
('CLP', 'USD', 0.00108),   -- Peso Chileno a Dólar
('ARS', 'USD', 0.0012),    -- Peso Argentino a Dólar (Blue/Referencial)
('EUR', 'USD', 1.085),     -- Euro a Dólar
('GBP', 'USD', 1.268),     -- Libra a Dólar
('MXN', 'USD', 0.058),     -- Peso Mexicano a Dólar
('BRL', 'USD', 0.201),     -- Real Brasileño a Dólar

-- De Dólar a Monedas Locales (Para compras internacionales)
('USD', 'CLP', 925.00),
('USD', 'ARS', 830.00),
('USD', 'EUR', 0.921),
('USD', 'GBP', 0.788),

-- Cruces Directos (Reto Pro)
('CLP', 'ARS', 1.12),      -- Para el fan chileno que va a Argentina
('GBP', 'EUR', 1.17);      -- Para el fan inglés que va a Italia

INSERT INTO staff_hierarchy (name, role, salary, manager_id, tour_id) VALUES
-- Staff para el Tour de AC/DC (tour_id 1)
('Angus Young Jr.', 'Tour Manager', 15000.00, NULL, 1),       -- El Jefe máximo
('Phil Carson', 'Stage Manager', 8000.00, 1, 1),             -- Reporta a Angus Jr.
('Danny O''Brian', 'Sound Engineer', 6000.00, 2, 1),         -- Reporta a Phil
('Stevie Van', 'Roadie', 3500.00, 2, 1),                     -- Reporta a Phil

-- Staff para el Tour de Metallica (tour_id 2)
('Cliff Burnstein', 'Global Manager', 20000.00, NULL, 2),     -- El Jefe máximo
('Peter Mensch', 'Tour Director', 12000.00, 5, 2),           -- Reporta a Cliff
('Big Mick', 'FOH Engineer', 9000.00, 6, 2),                 -- Reporta a Peter
('Justin Crew', 'Guitar Tech', 5000.00, 6, 2),               -- Reporta a Peter

-- Staff para Iron Maiden (tour_id 7)
('Rod Smallwood', 'Lead Manager', 18000.00, NULL, 7),
('Dickie Bell', 'Production Manager', 9500.00, 9, 7);        -- Reporta a Rod

INSERT INTO tour_history_archive (artist_id, venue_name, city, country, tour_date, start_time) VALUES
-- Registros para UNION (coinciden con la tabla actual)
(4, 'Live Aid - Wembley', 'London', 'UK', '1985-07-13', '18:41:00'),         -- Queen (Duplicado exacto)
(10, 'Pompeii Amphitheatre', 'Pompeii', 'Italy', '1971-10-04', '12:00:00'),  -- Pink Floyd (Duplicado exacto)

-- Conciertos Históricos Únicos (Siglo XX)
(3, 'Madison Square Garden', 'New York', 'USA', '1973-07-29', '20:30:00'),   -- Led Zeppelin (Noche distinta)
(8, 'Budokan', 'Tokyo', 'Japan', '1972-08-15', '19:00:00'),                  -- Deep Purple (Made in Japan)
(5, 'California Jam', 'Ontario', 'USA', '1974-04-06', '16:00:00'),           -- Black Sabbath
(1, 'Apollo Theatre', 'Glasgow', 'UK', '1978-04-30', '21:00:00'),            -- AC/DC (Bon Scott Era)
(6, 'Long Beach Arena', 'Los Angeles', 'USA', '1985-03-14', '20:00:00'),      -- Iron Maiden (Live After Death)
(10, 'Earls Court', 'London', 'UK', '1980-08-05', '20:00:00'),               -- Pink Floyd (The Wall Tour)

-- Datos con inconsistencias para enseñar limpieza
(99, 'Unknown Venue', 'Seattle', 'USA', '1989-01-01', NULL),                 -- Banda que ya no existe (ID 99)
(2, 'Old Stadium', 'Mexico City', 'Mexico', '1993-05-10', '21:00:00');       -- Metallica (Tour antiguo)
```
# 🎸 SQL Rock Academy – Roadmap de Ejercicios
**De Básico a Nivel Leyenda**

Este roadmap está diseñado para aprender SQL de forma progresiva, usando un dominio musical (bandas, fans, tours y staff) y cubriendo desde limpieza básica hasta CTEs recursivas y window functions.

---

## 🟢 Fase 1: Los Cimientos (Básico)
**Objetivo:** Dominar la selección, limpieza, formateo y ordenamiento de datos.

### Limpieza y Formato
1. **Limpieza de Nombres**  
   Selecciona todos los artistas, elimina espacios en blanco y asegúrate de que los nombres comiencen con mayúscula  
   *(TRIM, INITCAP)*

2. **El Problema de los NULL**  
   Muestra el nombre completo de los fans.  
   Si no tienen apellido, mostrar `"Sin Apellido"`  
   *(COALESCE, CONCAT)*

3. **Búsqueda por Patrón**  
   Encuentra artistas que contengan la palabra `Led` en su nombre, considerando errores de tipeo como `"Zeppelinn"`  
   *(LIKE, ILIKE)*

4. **Filtrado por Género**  
   Lista bandas de `'Hard Rock'` o `'Heavy Metal'` formadas antes de 1980

---

### Nuevos Retos de Limpieza y Orden
5. **El Buscador de "Guns"**  
   Encuentra a *Guns N' Roses* manejando correctamente la comilla simple (`'`)

6. **Formatos de Salida**  
   Genera un reporte:  
   `"El fan [Nombre] vive en [País] y su moneda es [Moneda]"`  
   *(Concatenación con `||`)*

7. **Ordenamiento Priorizado**  
   Lista fans de USA primero y luego el resto de los países en orden alfabético  
   *(CASE dentro de ORDER BY)*

---

## 🟡 Fase 2: Agregaciones y Grupos (Intermedio)
**Objetivo:** Resumir información y generar métricas.

### Agregaciones Básicas
1. **Conteo de Fans**  
   ¿Cuántos fans hay por país y cuál es el promedio de dinero en sus billeteras?  
   *(COUNT, AVG, GROUP BY)*

2. **Bandas por Época**  
   Cuenta cuántas bandas se formaron por década  
   *(FLOOR, divisiones matemáticas)*

3. **Sueldos del Staff**  
   Calcula el costo total de la nómina por rol  
   *(SUM, ORDER BY)*

---

### Nuevos Retos de Analítica
4. **Filtro de Grupos (HAVING)**  
   Muestra solo países con más de 2 fans registrados

5. **Rango de Salarios**  
   Obtén salario máximo, mínimo y brecha salarial del staff del tour de Metallica

6. **Frecuencia de Tours**  
   ¿Cuál es el año con más conciertos en toda la historia (actual + archivo)?

---

## 🔵 Fase 3: El Poder de los JOINS (Relacional)
**Objetivo:** Conectar todo el ecosistema de datos.

### Joins Clásicos
1. **Cartelera de Conciertos**  
   Nombre de la banda, ciudad y fecha de conciertos realizados en 2024

2. **Mapa de Precios**  
   Une conciertos con precios y muestra cuánto cuesta la entrada *Premium* por país

3. **¿Quién manda a quién? (Self Join)**  
   Muestra empleado y su jefe directo  
   Si no tiene jefe, mostrar `"Top Manager"`

---

### Nuevos Retos de Cruces
4. **Fans sin Conciertos**  
   Fans que viven en países donde no hay ningún tour  
   *(LEFT JOIN + IS NULL)*

5. **Staff Huérfano**  
   Empleados sin tour asignado actualmente

6. **Join Multidireccional**  
   Vista completa:  
   **Banda → Tour → Precios → Staff** (4 tablas)

---

## 🔴 Fase 4: Lógica de Negocio y Conversión (Avanzado)
**Objetivo:** Pensar como ingeniero de datos y analista financiero.

### Conversión y Validación
1. **El Desafío Internacional**  
   Convertir el precio de una entrada *Cancha* de Argentina (ARS) a CLP  
   *(Tabla exchange_rates)*

2. **Validación de Compra**  
   Booleano que indique si a cada fan le alcanza para comprar una entrada *Premium* en su país

3. **Estandarización Global**  
   Convertir todos los wallets y precios a USD

---

### Nuevos Retos Financieros
4. **Impuesto al Rock**  
   Aplica IVA 19% solo a conciertos realizados en Chile

5. **Arbitraje de Moneda**  
   ¿Existe diferencia real entre la entrada *Premium* de AC/DC (Chile) y Metallica (Argentina) en USD?

6. **Alerta de Saldo Bajo**  
   Fans que, tras comprar entrada *Cancha*, quedan con menos de 5 USD

---

## 🟣 Fase 5: CTEs y Window Functions (Nivel Pro)
**Objetivo:** Analítica avanzada y queries profesionales.

### Window Functions
1. **Top 3 Staff por Tour**  
   Ranking de los empleados mejor pagados por tour  
   *(RANK OVER PARTITION BY)*

2. **Promedio Móvil de Giras**  
   Promedio de precios de tickets por banda a lo largo del tiempo

3. **Salario vs Promedio**  
   Compara salario individual vs promedio del rol  
   *(AVG OVER PARTITION BY role)*

4. **First & Last Concert**  
   Primer y último concierto de Pink Floyd sin usar MIN/MAX  
   *(FIRST_VALUE, LAST_VALUE)*

---

### CTEs
5. **Análisis de Capacidad de Compra**  
   - CTE `Fans_en_USD`  
   - CTE `Tickets_en_USD`  
   Unirlas para ver quién puede comprar qué

6. **CTE Recursiva (Opcional – Nivel Leyenda)**  
   Cadena completa de mando desde un Roadie hasta el Tour Manager

---

## ⚫ Fase 6: Uniones y Fechas (Full Stack)
**Objetivo:** Integrar historia, tiempo y proyección futura.

### UNION & Fechas
1. **Cronología Definitiva**  
   Unir conciertos actuales + archivo, sin duplicados, ordenados por fecha

2. **Diferencia de Años**  
   Años entre el primer concierto histórico y el próximo concierto de 2025

3. **Segmentación Horaria**  
   Clasificar conciertos en Mañana / Tarde / Noche

---

### Nuevos Retos Temporales
4. **Identificación de Origen**  
   Agregar columna `'DATO_HISTORICO'` o `'DATO_ACTUAL'` al unir tablas

5. **Conciertos de Fin de Semana**  
   Filtrar shows en sábado o domingo  
   *(EXTRACT DOW / TO_CHAR)*

6. **Próximo Show**  
   Para cada banda, mostrar su concierto más cercano a `CURRENT_DATE`

---

🎯 **Resultado final:**  
Si alguien completa este roadmap, puede decir sin miedo:

> *“Sé SQL de verdad.”*

- Te comunicas de forma clara capaz de llegar a mucha gente
- Tiendo a trabajar bien en equipo
- Tiendo a permanecer calmadao bajo presión
- Que soy capaz de entregar lo que se me pide con altos estandares de trabajo
- Que tiendo a estar abierto a nueva información y encontrar formas de utilizarlo
- Que tiendo a ailar areas de problemas y usar tecnicas efectivas para resolverlas
- Que soy rápido en cambiar de tareas
- Que facilmente proceso y manipulo información númerica
- Que soy capaz de resolver problemas a través de la mayoría de las situaciones
- Que tengo la capacidad de leer y procesar información compleja en mi mente

Comunicación
Colaboración
Temple
Calidad
