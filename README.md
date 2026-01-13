# 🛒 Star Schema E-Commerce - Guía Rápida

| Autor            | Fecha        | Día |
|------------------|--------------|----------|
| **Carlos Vásquez** |12 Enero 2026 | 1| 

## 📋 Requisitos
- PostgreSQL instalado
- Comando: `psql -U postgres`

---

## 🚀 PASO 1: Setup Inicial

```sql
-- Crear y conectar
CREATE DATABASE ecommerce_analytics
    WITH ENCODING = 'UTF8';
\c ecommerce_analytics
```

---

## 📊 PASO 2: Star Schema Completo

```sql
-- DIMENSIÓN TIEMPO
CREATE TABLE dim_tiempo (
    id SERIAL PRIMARY KEY,
    fecha DATE UNIQUE,
    dia INTEGER,
    mes INTEGER,
    nombre_mes VARCHAR(20),
    trimestre INTEGER,
    anio INTEGER,
    dia_semana VARCHAR(10),
    numero_semana INTEGER,
    festivo BOOLEAN,
    temporada VARCHAR(20),
    fin_semana BOOLEAN,
    dia_habil BOOLEAN
);

-- DIMENSIÓN CLIENTE
CREATE TABLE dim_cliente (
    id SERIAL PRIMARY KEY,
    id_cliente_natural INTEGER,
    nombre VARCHAR(100),
    email VARCHAR(100),
    fecha_registro DATE,
    segmento_valor VARCHAR(20),
    segmento_comportamiento VARCHAR(30),
    edad INTEGER,
    genero VARCHAR(10),
    ciudad VARCHAR(50),
    region VARCHAR(50),
    pais VARCHAR(50),
    frecuencia_compras_mensual DECIMAL(4,1),
    valor_promedio_compra DECIMAL(10,2),
    ultima_compra DATE,
    activo BOOLEAN
);

-- DIMENSIÓN PRODUCTO
CREATE TABLE dim_producto (
    id SERIAL PRIMARY KEY,
    sku VARCHAR(20) UNIQUE,
    nombre VARCHAR(100),
    descripcion TEXT,
    categoria VARCHAR(50),
    subcategoria VARCHAR(50),
    marca VARCHAR(50),
    precio_lista DECIMAL(10,2),
    costo DECIMAL(10,2),
    margen DECIMAL(5,2),
    stock_actual INTEGER,
    disponible BOOLEAN,
    fecha_lanzamiento DATE
);

-- DIMENSIÓN GEOGRAFÍA
CREATE TABLE dim_geografia (
    id SERIAL PRIMARY KEY,
    codigo_postal VARCHAR(10),
    ciudad VARCHAR(50),
    provincia VARCHAR(50),
    region VARCHAR(50),
    pais VARCHAR(50),
    zona_horaria VARCHAR(10)
);

-- DIMENSIÓN CANAL
CREATE TABLE dim_canal_adquisicion (
    id SERIAL PRIMARY KEY,
    nombre_canal VARCHAR(50),
    tipo_canal VARCHAR(20),
    costo_adquisicion DECIMAL(8,2),
    roi_promedio DECIMAL(5,2),
    tasa_conversion DECIMAL(5,2),
    activo BOOLEAN
);

-- TABLA DE HECHOS
CREATE TABLE hechos_ventas (
    id_venta SERIAL PRIMARY KEY,
    id_tiempo INTEGER REFERENCES dim_tiempo(id),
    id_cliente INTEGER REFERENCES dim_cliente(id),
    id_producto INTEGER REFERENCES dim_producto(id),
    id_canal INTEGER REFERENCES dim_canal_adquisicion(id),
    id_geografia INTEGER REFERENCES dim_geografia(id),
    
    cantidad INTEGER,
    precio_unitario DECIMAL(10,2),
    descuento_aplicado DECIMAL(10,2),
    costo_envio DECIMAL(10,2),
    impuestos DECIMAL(10,2),
    
    total_bruto DECIMAL(10,2),
    total_neto DECIMAL(10,2),
    margen_contribucion DECIMAL(10,2),
    
    primera_compra BOOLEAN,
    compra_recurrente BOOLEAN,
    cliente_vip BOOLEAN
);
```

---

## 📥 PASO 3: Cargar Datos de Ejemplo

```sql
-- 1. DIMENSIÓN TIEMPO (genera todo 2024 automáticamente)
INSERT INTO dim_tiempo (fecha, dia, mes, nombre_mes, trimestre, anio, dia_semana, numero_semana, festivo, temporada, fin_semana, dia_habil)
SELECT 
    fecha::DATE,
    EXTRACT(DAY FROM fecha)::INTEGER,
    EXTRACT(MONTH FROM fecha)::INTEGER,
    CASE EXTRACT(MONTH FROM fecha)
        WHEN 1 THEN 'Enero' WHEN 2 THEN 'Febrero' WHEN 3 THEN 'Marzo'
        WHEN 4 THEN 'Abril' WHEN 5 THEN 'Mayo' WHEN 6 THEN 'Junio'
        WHEN 7 THEN 'Julio' WHEN 8 THEN 'Agosto' WHEN 9 THEN 'Septiembre'
        WHEN 10 THEN 'Octubre' WHEN 11 THEN 'Noviembre' WHEN 12 THEN 'Diciembre'
    END,
    EXTRACT(QUARTER FROM fecha)::INTEGER,
    EXTRACT(YEAR FROM fecha)::INTEGER,
    CASE EXTRACT(DOW FROM fecha)
        WHEN 0 THEN 'Domingo' WHEN 1 THEN 'Lunes' WHEN 2 THEN 'Martes'
        WHEN 3 THEN 'Miércoles' WHEN 4 THEN 'Jueves' WHEN 5 THEN 'Viernes' WHEN 6 THEN 'Sábado'
    END,
    EXTRACT(WEEK FROM fecha)::INTEGER,
    FALSE,
    CASE 
        WHEN EXTRACT(MONTH FROM fecha) IN (12,1,2) THEN 'Verano'
        WHEN EXTRACT(MONTH FROM fecha) IN (3,4,5) THEN 'Otoño'
        WHEN EXTRACT(MONTH FROM fecha) IN (6,7,8) THEN 'Invierno'
        ELSE 'Primavera'
    END,
    CASE WHEN EXTRACT(DOW FROM fecha) IN (0,6) THEN TRUE ELSE FALSE END,
    CASE WHEN EXTRACT(DOW FROM fecha) BETWEEN 1 AND 5 THEN TRUE ELSE FALSE END
FROM generate_series('2024-01-01'::DATE, '2024-12-31'::DATE, '1 day'::INTERVAL) AS fecha;

-- 2. CLIENTES
INSERT INTO dim_cliente VALUES
(1, 1, 'José Pérez', 'jose@email.com', '2024-01-15', 'Oro', 'Recurrente', 32, 'Masculino', 'Santiago', 'Región Metropolitana', 'Chile', 2.5, 450000, '2024-11-15', TRUE),
(2, 2, 'María González', 'maria@email.com', '2024-03-20', 'Plata', 'Nuevo', 28, 'Femenino', 'Viña del Mar', 'Región de Valparaíso', 'Chile', 1.2, 350000, '2024-11-16', TRUE),
(3, 3, 'Ramón Rodríguez', 'ramon@email.com', '2023-06-10', 'Platino', 'VIP', 45, 'Masculino', 'Santiago', 'Región Metropolitana', 'Chile', 4.0, 850000, '2024-12-05', TRUE);

-- 3. PRODUCTOS
INSERT INTO dim_producto VALUES
(1, 'SM-S21-BLK', 'Samsung Galaxy S21', 'Teléfono inteligente premium', 'Electrónica', 'Smartphones', 'Samsung', 699000, 500000, 28.5, 25, TRUE, '2024-01-01'),
(2, 'NK-AIR-WHT', 'Nike Air Max', 'Zapatillas deportivas', 'Ropa', 'Calzado', 'Nike', 89000, 60000, 32.6, 35, TRUE, '2024-02-15'),
(3, 'APL-IP13-BLU', 'iPhone 13', 'Smartphone Apple', 'Electrónica', 'Smartphones', 'Apple', 899000, 650000, 27.7, 18, TRUE, '2024-01-05');

-- 4. GEOGRAFÍA
INSERT INTO dim_geografia VALUES
(1, '8320000', 'Santiago', 'Santiago', 'Región Metropolitana', 'Chile', 'UTC-3'),
(2, '2520000', 'Viña del Mar', 'Valparaíso', 'Región de Valparaíso', 'Chile', 'UTC-3');

-- 5. CANALES
INSERT INTO dim_canal_adquisicion VALUES
(1, 'Google Ads', 'Pago', 1500, 3.5, 2.8, TRUE),
(2, 'Facebook Ads', 'Pago', 1200, 3.2, 3.1, TRUE),
(3, 'SEO Orgánico', 'Orgánico', 0, 8.5, 4.2, TRUE);

-- 6. HECHOS DE VENTAS (usa subconsultas para obtener IDs correctos)
INSERT INTO hechos_ventas 
SELECT 1, t.id, 1, 1, 1, 1, 1, 699000, 50000, 5000, 132810, 699000, 786810, 189634, FALSE, TRUE, TRUE
FROM dim_tiempo t WHERE t.fecha = '2024-11-15'
UNION ALL
SELECT 2, t.id, 2, 2, 2, 2, 1, 89000, 0, 3000, 17480, 89000, 109480, 26347, TRUE, FALSE, FALSE
FROM dim_tiempo t WHERE t.fecha = '2024-11-16'
UNION ALL
SELECT 3, t.id, 3, 3, 3, 1, 1, 899000, 100000, 0, 151905, 899000, 950905, 228977, FALSE, TRUE, TRUE
FROM dim_tiempo t WHERE t.fecha = '2024-11-18';

-- Verificar que todo se cargó correctamente
SELECT 'dim_tiempo' AS tabla, COUNT(*) AS registros FROM dim_tiempo
UNION ALL SELECT 'dim_cliente', COUNT(*) FROM dim_cliente
UNION ALL SELECT 'dim_producto', COUNT(*) FROM dim_producto
UNION ALL SELECT 'dim_geografia', COUNT(*) FROM dim_geografia
UNION ALL SELECT 'dim_canal_adquisicion', COUNT(*) FROM dim_canal_adquisicion
UNION ALL SELECT 'hechos_ventas', COUNT(*) FROM hechos_ventas;
```

---

## ⚡ PASO 4: Comparación de Consultas

```sql
-- Activar medición de tiempo
\timing

-- Consulta en Star Schema (SIMPLE Y RÁPIDO)
SELECT 
    dc.nombre as cliente,
    dp.nombre as producto,
    dp.categoria,
    SUM(hv.total_neto) as total
FROM hechos_ventas hv
JOIN dim_tiempo dt ON hv.id_tiempo = dt.id
JOIN dim_cliente dc ON hv.id_cliente = dc.id
JOIN dim_producto dp ON hv.id_producto = dp.id
WHERE dt.anio = 2024
GROUP BY dc.nombre, dp.nombre, dp.categoria
ORDER BY total DESC;
```
![ejemplo](ejemplo.png)
---

## 🚀 PASO 5: Optimizaciones

```sql
-- Crear índices para mejorar performance
CREATE INDEX idx_hechos_tiempo ON hechos_ventas(id_tiempo);
CREATE INDEX idx_hechos_cliente ON hechos_ventas(id_cliente);
CREATE INDEX idx_hechos_producto ON hechos_ventas(id_producto);

-- Vista materializada para reportes frecuentes
CREATE MATERIALIZED VIEW mv_ventas_mensuales AS
SELECT 
    dt.anio,
    dt.mes,
    dt.nombre_mes,
    dp.categoria,
    SUM(hv.total_neto) AS ventas_totales,
    COUNT(*) AS num_transacciones,
    AVG(hv.total_neto) AS ticket_promedio
FROM hechos_ventas hv
JOIN dim_tiempo dt ON hv.id_tiempo = dt.id
JOIN dim_producto dp ON hv.id_producto = dp.id
GROUP BY dt.anio, dt.mes, dt.nombre_mes, dp.categoria;

-- Consultar vista (MUY RÁPIDO)
SELECT * FROM mv_ventas_mensuales WHERE anio = 2024 ORDER BY ventas_totales DESC;
```
![ejemplo2](ejemplo2.png)
---

## 📊 PASO 6: Consultas Analíticas

```sql
-- Top 5 productos más vendidos
SELECT 
    dp.nombre,
    dp.categoria,
    SUM(hv.total_neto) as ventas,
    COUNT(*) as num_ventas
FROM hechos_ventas hv
JOIN dim_producto dp ON hv.id_producto = dp.id
GROUP BY dp.nombre, dp.categoria
ORDER BY ventas DESC
LIMIT 5;
```
![ejemplo3](ejemplo3.png)
```sql
-- Performance por canal de adquisición
SELECT 
    dca.nombre_canal,
    dca.tipo_canal,
    COUNT(*) as conversiones,
    TO_CHAR(SUM(hv.total_neto), 'FM$999,999,999') as revenue,
    ROUND(SUM(hv.total_neto) / NULLIF(dca.costo_adquisicion * COUNT(*), 0), 2) as roi
FROM hechos_ventas hv
JOIN dim_canal_adquisicion dca ON hv.id_canal = dca.id
GROUP BY dca.nombre_canal, dca.tipo_canal, dca.costo_adquisicion
ORDER BY SUM(hv.total_neto) DESC;


-- Ventas por mes y categoría
SELECT 
    dt.nombre_mes,
    dp.categoria,
    TO_CHAR(SUM(hv.total_neto), 'FM$999,999,999') as ventas
FROM hechos_ventas hv
JOIN dim_tiempo dt ON hv.id_tiempo = dt.id
JOIN dim_producto dp ON hv.id_producto = dp.id
GROUP BY dt.mes, dt.nombre_mes, dp.categoria
ORDER BY dt.mes, ventas DESC;
```

---

## ✅ Resumen

**Star Schema = Análisis Simple y Rápido**
- ✔️ Menos JOINS (3-4 vs 8-10 en normalizado)
- ✔️ Consultas intuitivas y legibles
- ✔️ Performance superior (10-100x más rápido)
- ✔️ Optimizado para Business Intelligence

**Trade-off**: Más espacio de almacenamiento vs Mayor velocidad de consulta

## Evidencia Prueba curso
![Evidencia](prueba.png)
---
