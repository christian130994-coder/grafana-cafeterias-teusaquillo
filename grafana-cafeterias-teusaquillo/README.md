# Actividad 5. Dashboard analítico en Grafana sobre Amazon RDS

**Herramientas de Computación en la Nube** · Maestría en Analítica Aplicada · Universidad de La Sabana
Autor: Christian Fabián Jaimes Toscano

## Objetivo

Cerrar el pipeline de cafeterías de Teusaquillo construido en las actividades
anteriores con una capa de visualización: un dashboard en Grafana, desplegado
en Amazon EC2, que consulta directamente la base Amazon RDS PostgreSQL y
convierte los 46 establecimientos en información para decidir dónde abrir,
competir o diferenciarse.

## Arquitectura

```
Foursquare API → S3 (zona raw) → AWS Lambda (UPSERT) → Amazon RDS PostgreSQL
                                                              │  usuario de solo lectura, SSL
                                                              ▼
                                    Amazon EC2 (Amazon Linux 2023) + Docker + Grafana
                                                              │  puerto 3000, visor anónimo
                                                              ▼
                                                     Dashboard (8 paneles)
```

## Contenido

| Ruta | Descripción |
|---|---|
| `sql/01_ddl.sql` | Tabla `cafeterias_teusaquillo` con tipos, restricciones e índices (Actividad 3) |
| `sql/02_carga.sql` | Carga idempotente del CSV (misma transformación de la Lambda, UPSERT) |
| `sql/03_usuario_lectura.sql` | Usuario `grafana_reader` de solo lectura (mínimo privilegio) |
| `sql/04_consultas_dashboard.sql` | Las 7 consultas del dashboard, documentadas |
| `grafana/dashboards/cafeterias.json` | Dashboard listo para importar |
| `grafana/provisioning/` | Conexión a RDS y carga automática del dashboard (sin credenciales en el código) |
| `infra/instalar.sh` | Instalación completa en la EC2 en un solo comando |
| `data/cafeterias_teusaquillo_limpio.csv` | 46 cafeterías validadas (zona raw, Actividad 2) |
| `evidencias/` | Capturas de la entrega |

## Paneles

1. Indicadores clave (total, % multicategoría, distancia mediana, % de celdas con oferta)
2. Mapa de cafeterías por categoría (geomap)
3. Densidad por celda de ~500 m (geomap)
4. Histograma de distancias
5. Frecuencia de categorías desde JSONB
6. Celdas por nivel de competencia
7. Ranking de zonas y oportunidades (consulta compleja: CTE, `generate_series`, `LEFT JOIN`, funciones de ventana)
8. Inventario de establecimientos (tabla)

## Despliegue

En la EC2 (Amazon Linux 2023), con la base RDS ya creada:

```bash
sudo dnf install -y git
git clone https://github.com/<tu-usuario>/grafana-cafeterias-teusaquillo.git
cd grafana-cafeterias-teusaquillo
sudo bash infra/instalar.sh
```

## Seguridad y FinOps

- Grafana se conecta con `grafana_reader`, que solo tiene `SELECT` sobre la tabla.
- Las claves se piden por teclado; el archivo de entorno queda en la EC2 con permisos `600` y está excluido del repositorio.
- RDS no es pública: solo acepta el puerto 5432 desde el grupo de seguridad de la EC2, con SSL obligatorio.
- El puerto 22 se abre solo durante la instalación; el evaluador entra como visor anónimo de solo lectura.
- Al finalizar se termina la EC2 y se elimina o detiene la base RDS.
