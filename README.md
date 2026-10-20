<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Cotizador de Fletes | Servifácil</title>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/leaflet@1.9.4/dist/leaflet.css">
<style>
:root {
  --bg: #111214;
  --panel: #16171a;
  --card: #1c1e22;
  --card-2: #22252a;
  --card-3: #292c32;
  --border: #2d3036;
  --border-2: #3a3e45;
  --text: #e9eaec;
  --muted: #a1a6ae;
  --faint: #6f757e;
  --accent: #4a9eff;
  --accent-strong: #2f80e8;
  --accent-soft: rgba(74, 158, 255, .13);
  --warn: #f2b01e;
  --warn-soft: rgba(242, 176, 30, .10);
  --warn-border: rgba(242, 176, 30, .45);
  --ok: #22c55e;
  --ok-soft: rgba(34, 197, 94, .13);
  --bad: #ef4444;
  --bad-soft: rgba(239, 68, 68, .13);
  --violet: #a78bfa;
  --shadow: 0 10px 30px rgba(0, 0, 0, .45);
  --radius: 10px;
  --radius-sm: 7px;
  --font: "Segoe UI", system-ui, -apple-system, Roboto, Arial, sans-serif;
  --mono: ui-monospace, "Cascadia Mono", Consolas, monospace;
  color-scheme: dark;
}
:root[data-theme="light"] {
  --bg: #f3f4f6; --panel: #ffffff; --card: #ffffff; --card-2: #f6f7f9; --card-3: #eceef2;
  --border: #e1e4e8; --border-2: #cfd4da; --text: #15181d; --muted: #59606a; --faint: #8a919b;
  --accent: #1f6fd1; --accent-strong: #165bb0; --accent-soft: rgba(31, 111, 209, .10);
  --warn: #a96b00; --warn-soft: rgba(242, 176, 30, .14); --warn-border: rgba(169, 107, 0, .45);
  --ok: #15803d; --ok-soft: rgba(21, 128, 61, .10); --bad: #c62828; --bad-soft: rgba(198, 40, 40, .09);
  --violet: #6d4fd8; --shadow: 0 10px 30px rgba(15, 23, 42, .14);
  color-scheme: light;
}
* { box-sizing: border-box; }
html, body { margin: 0; height: 100%; background: var(--bg); color: var(--text); font-family: var(--font); font-size: 14px; }
body { overflow: hidden; }
a { color: var(--accent); }
button, input, select, textarea { font: inherit; color: inherit; }
svg.i { width: 16px; height: 16px; flex: 0 0 auto; stroke: currentColor; fill: none; stroke-width: 1.9; stroke-linecap: round; stroke-linejoin: round; vertical-align: -3px; }
.oculto { display: none !important; }
.mono { font-family: var(--mono); }
.muted { color: var(--muted); }
.faint { color: var(--faint); }
.der { text-align: right; }
.nowrap { white-space: nowrap; }

/* ---------- Estructura ---------- */
.app { display: grid; grid-template-columns: 228px 1fr; height: 100vh; }
.app.colapsada { grid-template-columns: 60px 1fr; }
.sidebar { background: var(--panel); border-right: 1px solid var(--border); display: flex; flex-direction: column; min-height: 0; }
.marca { display: flex; align-items: center; gap: 10px; padding: 14px 14px 12px; border-bottom: 1px solid var(--border); min-height: 62px; }
.marca img { width: 34px; height: 34px; object-fit: contain; border-radius: 6px; background: #fff; }
.marca b { font-size: 14px; line-height: 1.15; }
.marca small { display: block; color: var(--faint); font-size: 11px; }
.nav { padding: 10px 8px; overflow-y: auto; flex: 1; }
.nav .grupo { color: var(--faint); font-size: 10.5px; letter-spacing: .06em; text-transform: uppercase; margin: 12px 10px 6px; }
.nav a { display: flex; align-items: center; gap: 11px; padding: 8px 10px; border-radius: var(--radius-sm); color: var(--muted); text-decoration: none; margin-bottom: 2px; white-space: nowrap; }
.nav a:hover { background: var(--card-2); color: var(--text); }
.nav a.activo { background: var(--accent-soft); color: var(--accent); }
.nav a svg.i { width: 18px; height: 18px; }
.app.colapsada .nav .grupo, .app.colapsada .nav a span, .app.colapsada .marca div { display: none; }
.app.colapsada .nav a { justify-content: center; }
.pie-sidebar { padding: 10px 14px; border-top: 1px solid var(--border); color: var(--faint); font-size: 11px; font-style: italic; }
.app.colapsada .pie-sidebar { display: none; }

.principal { display: flex; flex-direction: column; min-width: 0; min-height: 0; }
.topbar { display: flex; align-items: center; gap: 12px; padding: 0 18px; height: 56px; border-bottom: 1px solid var(--border); background: var(--panel); flex: 0 0 auto; }
.topbar h1 { font-size: 15px; font-weight: 600; margin: 0; display: flex; align-items: center; gap: 8px; }
.topbar h1 .sep { color: var(--faint); font-weight: 400; }
.topbar .espacio { flex: 1; }
.estado-guardado { display: inline-flex; align-items: center; gap: 6px; font-size: 12px; padding: 4px 10px; border-radius: 999px; background: var(--card-2); color: var(--muted); border: 1px solid var(--border); cursor: default; }
.estado-guardado .punto { width: 8px; height: 8px; border-radius: 50%; background: var(--ok); }
.estado-guardado.pendiente { cursor: pointer; color: var(--warn); border-color: var(--warn-border); background: var(--warn-soft); }
.estado-guardado.pendiente .punto { background: var(--warn); }
.usuario-btn { display: flex; align-items: center; gap: 8px; background: none; border: 1px solid transparent; padding: 4px 8px; border-radius: 999px; cursor: pointer; }
.usuario-btn:hover { border-color: var(--border); }
.avatar { width: 28px; height: 28px; border-radius: 50%; background: var(--card-3); display: grid; place-items: center; font-weight: 700; font-size: 12px; color: var(--muted); }
#vista { flex: 1; overflow: auto; padding: 20px 22px 40px; min-height: 0; }

/* ---------- Controles ---------- */
.btn { display: inline-flex; align-items: center; justify-content: center; gap: 7px; padding: 7px 13px; border-radius: 8px; border: 1px solid var(--border-2); background: var(--card-2); color: var(--text); cursor: pointer; font-weight: 500; white-space: nowrap; line-height: 1.2; }
.btn:hover { background: var(--card-3); }
.btn:disabled { opacity: .5; cursor: not-allowed; }
.btn.primario { background: var(--accent); border-color: var(--accent); color: #fff; }
.btn.primario:hover { background: var(--accent-strong); }
.btn.suave { background: var(--accent-soft); border-color: transparent; color: var(--accent); }
.btn.fantasma { background: transparent; border-color: transparent; color: var(--accent); padding-left: 6px; padding-right: 6px; }
.btn.fantasma:hover { background: var(--accent-soft); }
.btn.peligro { color: var(--bad); }
.btn.chico { padding: 4px 9px; font-size: 12.5px; border-radius: 7px; }
.btn.icono { padding: 6px; }
.btn.bloque { width: 100%; }
.campo { display: flex; flex-direction: column; gap: 5px; min-width: 0; }
.campo > label, .lbl { font-size: 12.5px; color: var(--muted); font-weight: 500; }
.campo .ayuda { font-size: 11.5px; color: var(--faint); font-style: italic; }
input[type=text], input[type=number], input[type=email], input[type=date], input[type=month], input[type=search], input[type=tel], input[type=url], input[type=password], select, textarea {
  background: var(--card-2); border: 1px solid var(--border-2); border-radius: 8px; padding: 7px 10px; width: 100%; min-width: 0; outline: none;
}
input:focus, select:focus, textarea:focus { border-color: var(--accent); box-shadow: 0 0 0 3px var(--accent-soft); }
input[type=number] { text-align: right; }
input.invalido { border-color: var(--bad); }
textarea { resize: vertical; min-height: 64px; }
input[type=checkbox], input[type=radio] { accent-color: var(--accent); width: 15px; height: 15px; margin: 0; }
.con-prefijo { position: relative; }
.con-prefijo > span { position: absolute; left: 10px; top: 50%; transform: translateY(-50%); color: var(--faint); pointer-events: none; font-size: 13px; }
.con-prefijo > input { padding-left: 24px; }
.con-sufijo { position: relative; }
.con-sufijo > span { position: absolute; right: 10px; top: 50%; transform: translateY(-50%); color: var(--faint); pointer-events: none; font-size: 12.5px; }
.con-sufijo > input { padding-right: 34px; }
.stepper { display: flex; align-items: center; border: 1px solid var(--border-2); border-radius: 8px; background: var(--card-2); overflow: hidden; }
.stepper button { background: none; border: 0; color: var(--accent); width: 34px; height: 32px; cursor: pointer; font-size: 18px; line-height: 1; }
.stepper button:hover { background: var(--card-3); }
.stepper input { border: 0 !important; box-shadow: none !important; text-align: center !important; background: transparent; width: 70px; padding: 6px 2px; }
.switch { position: relative; display: inline-flex; align-items: center; gap: 9px; cursor: pointer; user-select: none; }
.switch input { position: absolute; opacity: 0; width: 0; height: 0; }
.switch .pista { width: 34px; height: 19px; border-radius: 999px; background: var(--card-3); border: 1px solid var(--border-2); position: relative; transition: .15s; flex: 0 0 auto; }
.switch .pista::after { content: ""; position: absolute; top: 2px; left: 2px; width: 13px; height: 13px; border-radius: 50%; background: var(--muted); transition: .15s; }
.switch input:checked + .pista { background: var(--accent); border-color: var(--accent); }
.switch input:checked + .pista::after { left: 17px; background: #fff; }
.switch input:focus-visible + .pista { box-shadow: 0 0 0 3px var(--accent-soft); }
.grid-2 { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 12px; }
.grid-3 { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 12px; }
.grid-4 { display: grid; grid-template-columns: repeat(4, minmax(0, 1fr)); gap: 12px; }
.fila { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }
.fila.entre { justify-content: space-between; }
.pila { display: flex; flex-direction: column; gap: 12px; }

/* ---------- Tarjetas y pestañas ---------- */
.tarjeta { background: var(--card); border: 1px solid var(--border); border-radius: var(--radius); }
.tarjeta > .cab { display: flex; align-items: center; gap: 10px; padding: 14px 16px; border-bottom: 1px solid var(--border); }
.tarjeta > .cab h2, .tarjeta > .cab h3 { margin: 0; font-size: 15px; font-weight: 600; }
.tarjeta > .cuerpo { padding: 16px; }
.icono-cab { width: 30px; height: 30px; border-radius: 8px; display: grid; place-items: center; background: var(--accent-soft); color: var(--accent); flex: 0 0 auto; }
.pestanas { display: flex; gap: 4px; border-bottom: 1px solid var(--border); margin: -4px 0 18px; }
.pestanas button { background: none; border: 0; border-bottom: 2px solid transparent; padding: 9px 12px; color: var(--muted); cursor: pointer; display: inline-flex; gap: 7px; align-items: center; }
.pestanas button.activa { color: var(--accent); border-bottom-color: var(--accent); }
.barra-acciones { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; margin-bottom: 12px; }
.barra-acciones .espacio { flex: 1; }
.aviso { display: flex; gap: 10px; padding: 11px 13px; border-radius: 9px; border: 1px solid var(--warn-border); background: var(--warn-soft); color: var(--warn); font-size: 13px; line-height: 1.45; }
.aviso.info { border-color: rgba(74, 158, 255, .4); background: var(--accent-soft); color: var(--text); }
.aviso.info svg.i { color: var(--accent); }
.aviso.error { border-color: rgba(239, 68, 68, .45); background: var(--bad-soft); color: var(--bad); }
.aviso b { color: inherit; }
.aviso svg.i { margin-top: 1px; }
.vacio-estado { text-align: center; padding: 38px 16px; color: var(--muted); }
.vacio-estado .circulo { width: 40px; height: 40px; border-radius: 50%; background: var(--card-3); display: grid; place-items: center; margin: 0 auto 10px; }
.vacio-estado b { display: block; color: var(--text); margin-bottom: 4px; }
.badge { display: inline-flex; align-items: center; gap: 5px; padding: 2px 9px; border-radius: 999px; font-size: 11.5px; font-weight: 600; background: var(--card-3); color: var(--muted); white-space: nowrap; }
.badge.azul { background: var(--accent-soft); color: var(--accent); }
.badge.verde { background: var(--ok-soft); color: var(--ok); }
.badge.rojo { background: var(--bad-soft); color: var(--bad); }
.badge.ambar { background: var(--warn-soft); color: var(--warn); }

/* ---------- Tablas ---------- */
.tabla-wrap { border: 1px solid var(--border); border-radius: var(--radius); overflow: auto; background: var(--card); }
table.tabla { width: 100%; border-collapse: collapse; font-size: 13px; }
table.tabla th { position: sticky; top: 0; background: var(--card-2); color: var(--muted); font-weight: 600; text-align: left; padding: 9px 11px; border-bottom: 1px solid var(--border); white-space: nowrap; cursor: pointer; user-select: none; z-index: 1; }
table.tabla th.sin-orden { cursor: default; }
table.tabla td { padding: 8px 11px; border-bottom: 1px solid var(--border); vertical-align: middle; }
table.tabla tr:last-child td { border-bottom: 0; }
table.tabla tbody tr:hover td { background: rgba(255, 255, 255, .025); }
table.tabla td.num, table.tabla th.num { text-align: right; font-variant-numeric: tabular-nums; }
table.tabla td.editable { cursor: text; }
table.tabla td.editable:hover { outline: 1px dashed var(--border-2); outline-offset: -3px; }
table.tabla td input, table.tabla td select { padding: 4px 7px; font-size: 13px; }
table.tabla tfoot td { background: var(--card-2); font-weight: 600; }
table.tabla tr.click { cursor: pointer; }
.menu-fila { position: relative; }
.menu-pop { position: absolute; right: 0; top: 100%; background: var(--card-2); border: 1px solid var(--border-2); border-radius: 9px; box-shadow: var(--shadow); min-width: 170px; padding: 4px; z-index: 50; }
.menu-pop button { display: flex; gap: 8px; align-items: center; width: 100%; background: none; border: 0; text-align: left; padding: 7px 10px; border-radius: 6px; cursor: pointer; }
.menu-pop button:hover { background: var(--card-3); }

/* ---------- Modales y avisos emergentes ---------- */
.modal-fondo { position: fixed; inset: 0; background: rgba(0, 0, 0, .55); display: grid; place-items: center; z-index: 2000; padding: 16px; }
.modal { background: var(--card); border: 1px solid var(--border-2); border-radius: 12px; box-shadow: var(--shadow); width: min(560px, 100%); max-height: calc(100vh - 32px); display: flex; flex-direction: column; }
.modal.ancho { width: min(980px, 100%); }
.modal.medio { width: min(760px, 100%); }
.modal > .cab { display: flex; align-items: center; gap: 10px; padding: 12px 16px; border-bottom: 1px solid var(--border); background: var(--accent); color: #fff; border-radius: 11px 11px 0 0; }
.modal > .cab h3 { margin: 0; font-size: 15px; flex: 1; }
.modal > .cab button { background: none; border: 0; color: #fff; cursor: pointer; padding: 4px; }
.modal > .cuerpo { padding: 16px; overflow: auto; }
.modal > .pie { display: flex; justify-content: flex-end; gap: 8px; padding: 12px 16px; border-top: 1px solid var(--border); }
#toasts { position: fixed; right: 18px; bottom: 18px; display: flex; flex-direction: column; gap: 8px; z-index: 3000; }
.toast { background: var(--card-2); border: 1px solid var(--border-2); border-left: 4px solid var(--accent); padding: 10px 14px; border-radius: 9px; box-shadow: var(--shadow); max-width: 380px; font-size: 13px; animation: entra .18s ease-out; }
.toast.ok { border-left-color: var(--ok); }
.toast.error { border-left-color: var(--bad); }
.toast.warn { border-left-color: var(--warn); }
@keyframes entra { from { transform: translateY(8px); opacity: 0; } to { transform: none; opacity: 1; } }

/* ---------- Inicio ---------- */
.kpis { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 12px; }
.kpi { background: var(--card-2); border: 1px solid var(--border); border-radius: 9px; padding: 13px 14px; }
.kpi .t { color: var(--muted); font-size: 12.5px; display: flex; gap: 6px; align-items: center; }
.kpi .v { font-size: 24px; font-weight: 700; margin: 6px 0 2px; font-variant-numeric: tabular-nums; }
.kpi .s { color: var(--faint); font-size: 12px; }
.barra-estados { display: flex; height: 10px; border-radius: 999px; overflow: hidden; background: var(--card-3); gap: 2px; margin: 8px 0 12px; }
.barra-estados > div { height: 100%; }
.leyenda-estados { display: grid; grid-template-columns: repeat(5, minmax(0, 1fr)); gap: 10px; font-size: 12.5px; }
.leyenda-estados > div { display: flex; align-items: center; gap: 7px; }
.leyenda-estados b { margin-left: auto; font-variant-numeric: tabular-nums; }
.dot { width: 9px; height: 9px; border-radius: 3px; flex: 0 0 auto; }

/* ---------- Cotizador (mapa) ---------- */
#vistaCotizar { position: relative; flex: 1; min-height: 0; }
#mapa { position: absolute; inset: 0; background: #1b1d21; }
.panel-viaje { position: absolute; top: 12px; left: 12px; bottom: 12px; width: 372px; background: var(--panel); border: 1px solid var(--border); border-radius: 12px; box-shadow: var(--shadow); display: flex; flex-direction: column; z-index: 1100; }
.panel-viaje.minimizado { bottom: auto; }
.panel-viaje.minimizado .pv-cuerpo { display: none; }
.pv-cab { display: flex; align-items: center; gap: 8px; padding: 12px 14px 10px; border-bottom: 1px solid var(--border); }
.pv-cab h2 { font-size: 16px; margin: 0; flex: 1; }
.pv-cuerpo { overflow-y: auto; padding: 12px 14px 16px; flex: 1; }
.modos { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-bottom: 10px; }
.modos button { padding: 9px; border-radius: 9px; border: 1px solid var(--border-2); background: var(--card-2); color: var(--muted); cursor: pointer; display: inline-flex; gap: 7px; justify-content: center; align-items: center; font-weight: 500; }
.modos button.activo { border-color: var(--accent); color: var(--accent); background: var(--accent-soft); }
.subtitulo { font-size: 11px; letter-spacing: .06em; text-transform: uppercase; color: var(--faint); margin: 14px 0 8px; font-weight: 600; }
.paradas { display: flex; flex-direction: column; gap: 4px; }
.parada { border-radius: 9px; padding: 6px 4px; position: relative; }
.parada.arrastrando { opacity: .5; }
.parada .linea { display: flex; align-items: center; gap: 6px; }
.parada .asa { cursor: grab; color: var(--faint); padding: 2px; }
.parada .pin { color: var(--bad); }
.parada .pin.ok { color: var(--accent); }
.parada .buscador { flex: 1; position: relative; }
.parada .buscador input { padding-right: 30px; }
.parada .buscador .lupa { position: absolute; right: 8px; top: 50%; transform: translateY(-50%); color: var(--faint); }
.parada .meta { margin: 5px 0 0 30px; font-size: 12px; color: var(--muted); display: flex; flex-wrap: wrap; gap: 10px; align-items: center; }
.parada .meta label { display: inline-flex; gap: 5px; align-items: center; cursor: pointer; }
.parada .tiempos { margin: 4px 0 0 30px; font-size: 11.5px; color: var(--faint); text-decoration: underline dotted; cursor: pointer; }
.parada .editor-tiempos { margin: 6px 0 0 30px; display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
.sugerencias { position: absolute; left: 0; right: 0; top: calc(100% + 4px); background: var(--card-2); border: 1px solid var(--border-2); border-radius: 9px; box-shadow: var(--shadow); z-index: 700; max-height: 290px; overflow: auto; padding: 4px; }
.sugerencias button { display: flex; gap: 8px; align-items: flex-start; width: 100%; background: none; border: 0; padding: 7px 9px; border-radius: 6px; cursor: pointer; text-align: left; }
.sugerencias button:hover, .sugerencias button.foco { background: var(--card-3); }
.sugerencias .tipo { font-size: 10.5px; font-weight: 700; padding: 1px 6px; border-radius: 5px; background: var(--accent-soft); color: var(--accent); flex: 0 0 auto; margin-top: 1px; }
.sugerencias .tipo.tad { background: rgba(167, 139, 250, .16); color: var(--violet); }
.sugerencias .tipo.osm { background: var(--card-3); color: var(--muted); }
.sugerencias .sub { display: block; font-size: 11.5px; color: var(--faint); }
.resultados-ruta { margin-top: 14px; border: 1px solid var(--border); border-radius: 10px; background: var(--card); }
.resultados-ruta .cab { padding: 10px 12px; border-bottom: 1px solid var(--border); font-weight: 600; display: flex; gap: 8px; align-items: center; }
.resultados-ruta .f { display: flex; justify-content: space-between; align-items: center; padding: 8px 12px; border-bottom: 1px solid var(--border); }
.resultados-ruta .f:last-child { border-bottom: 0; }
.resultados-ruta .f span:first-child { color: var(--muted); text-decoration: underline dotted var(--faint); }
.resultados-ruta .f b { font-variant-numeric: tabular-nums; }
.resultados-ruta .total { cursor: pointer; }
.resultados-ruta .total:hover { background: var(--card-2); }
.leyenda-mapa { position: absolute; right: 12px; bottom: 26px; z-index: 1050; background: var(--panel); border: 1px solid var(--border); border-radius: 999px; padding: 6px 12px; font-size: 12px; display: flex; gap: 12px; align-items: center; box-shadow: var(--shadow); }
.leyenda-mapa i { display: inline-block; width: 22px; height: 0; border-top: 4px solid var(--accent); vertical-align: middle; margin-right: 5px; }
.leyenda-mapa i.v { border-top: 3px dashed #9aa0a8; }
.marcador-num { width: 26px; height: 26px; border-radius: 50% 50% 50% 0; transform: rotate(-45deg); background: var(--accent); border: 2px solid #fff; display: grid; place-items: center; box-shadow: 0 2px 6px rgba(0, 0, 0, .4); }
.marcador-num span { transform: rotate(45deg); color: #fff; font-weight: 700; font-size: 11.5px; }
.marcador-num.fin { background: #e5484d; }
.cursor-mapa .leaflet-container { cursor: crosshair; }

/* ---------- Detalle del viaje ---------- */
.panel-detalle { position: absolute; top: 12px; left: 12px; right: 12px; bottom: 12px; z-index: 1200; background: var(--panel); border: 1px solid var(--border-2); border-radius: 12px; box-shadow: var(--shadow); display: flex; flex-direction: column; min-width: 0; }
.pd-cab { display: flex; align-items: center; gap: 8px; background: var(--accent); color: #fff; padding: 8px 12px; border-radius: 11px 11px 0 0; }
.pd-cab h3 { margin: 0 8px 0 0; font-size: 15px; }
.pd-cab .tab { border: 0; border-radius: 8px; padding: 5px 11px; cursor: pointer; display: inline-flex; gap: 6px; align-items: center; font-weight: 600; font-size: 12.5px; background: rgba(255, 255, 255, .16); color: #fff; }
.pd-cab .tab.activa { background: #111214; color: #fff; }
.pd-cab .tab.verde { background: #1f9d55; }
.pd-cab .cerrar { margin-left: auto; background: none; border: 0; color: #fff; cursor: pointer; }
.pd-sub { display: flex; gap: 18px; padding: 8px 14px; border-bottom: 1px solid var(--border); color: var(--muted); font-size: 13px; flex-wrap: wrap; }
.pd-sub b { color: var(--text); }
.pd-cuerpo { flex: 1; overflow: auto; padding: 12px; }
.det-grid { display: grid; grid-template-columns: minmax(0, 1fr) minmax(0, 1.1fr) minmax(0, 1.1fr); gap: 12px; align-items: start; }
.det-col { display: flex; flex-direction: column; gap: 12px; min-width: 0; }
.bloque { background: var(--card); border: 1px solid var(--border); border-radius: 10px; }
.bloque > .t { display: flex; align-items: center; gap: 8px; padding: 10px 12px 6px; font-size: 12px; letter-spacing: .05em; text-transform: uppercase; color: var(--muted); font-weight: 600; }
.bloque > .t .ayuda-ico { margin-left: auto; color: var(--faint); cursor: help; }
.bloque .r { display: flex; justify-content: space-between; align-items: center; gap: 10px; padding: 4px 12px; font-size: 13px; }
.bloque .r > span:first-child { color: var(--muted); }
.bloque .r b, .bloque .r .v { font-variant-numeric: tabular-nums; font-weight: 600; white-space: nowrap; }
.bloque .r small { display: block; color: var(--faint); font-size: 11.5px; }
.bloque .r.tot { border-top: 1px solid var(--border); margin-top: 6px; padding-top: 9px; padding-bottom: 10px; }
.bloque .r.tot > span:first-child { color: var(--text); font-weight: 600; }
.bloque .r input[type=number] { width: 110px; padding: 5px 8px; }
.bloque .r .con-prefijo input, .bloque .r .con-sufijo input { width: 110px; }
.bloque .r.alerta > span:first-child { color: var(--warn); }
.bloque .r .marcas { display: inline-flex; gap: 6px; align-items: center; margin-right: 6px; }
.bloque .r .vacio-lbl { display: flex; align-items: center; gap: 8px; color: var(--muted); }
.bloque .pie-nota { padding: 0 12px 10px; font-size: 11.5px; color: var(--faint); font-style: italic; }
.chk-util { accent-color: var(--warn) !important; }
.total-iva { display: flex; justify-content: space-between; align-items: center; padding: 10px 12px; border-top: 1px solid var(--border); }
.total-iva span { font-size: 17px; font-weight: 700; }
.total-iva button { background: none; border: 0; color: var(--accent); font-size: 20px; font-weight: 700; cursor: pointer; display: inline-flex; gap: 6px; align-items: center; font-variant-numeric: tabular-nums; }
.total-iva button:hover { text-decoration: underline; }
.fila-carga { display: grid; grid-template-columns: minmax(0, 1.5fr) minmax(0, 1fr) minmax(0, 1fr); gap: 10px; align-items: end; padding: 10px 12px; }
.fila-carga .campo label { white-space: nowrap; display: inline-flex; gap: 4px; align-items: center; }
.fila-carga .con-sufijo input, .fila-carga input { width: 100%; }
.resumen-grid { display: grid; grid-template-columns: 300px 1fr; gap: 24px; align-items: center; padding: 10px 6px; }
.dona-wrap { position: relative; width: 280px; height: 280px; margin: 0 auto; }
.dona-wrap svg { width: 100%; height: 100%; }
.dona-wrap .centro { position: absolute; inset: 0; display: grid; place-items: center; text-align: center; pointer-events: none; }
.dona-wrap .centro b { font-size: 24px; display: block; font-variant-numeric: tabular-nums; }
.dona-tip { position: fixed; pointer-events: none; background: var(--card-3); border: 1px solid var(--border-2); border-radius: 7px; padding: 6px 9px; font-size: 12.5px; z-index: 4000; box-shadow: var(--shadow); }
.leyenda-dona { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 6px 22px; margin-top: 14px; font-size: 13px; }
.leyenda-dona > div { display: flex; align-items: center; gap: 8px; }
.leyenda-dona b { margin-left: auto; font-variant-numeric: tabular-nums; }

/* ---------- Formularios de catálogo ---------- */
.seccion-form { background: var(--card); border: 1px solid var(--border); border-radius: 10px; margin-bottom: 14px; }
.seccion-form > .t { padding: 11px 14px; border-bottom: 1px solid var(--border); font-size: 12px; text-transform: uppercase; letter-spacing: .05em; color: var(--muted); font-weight: 600; }
.seccion-form > .d { padding: 10px 14px 0; color: var(--faint); font-size: 12.5px; }
.seccion-form > .c { padding: 12px 14px 14px; }
.form-vehiculo { display: grid; grid-template-columns: minmax(0, 1fr) 320px; gap: 16px; align-items: start; }
.impacto { position: sticky; top: 0; }
.lista-mini { display: flex; flex-direction: column; gap: 8px; }
.lista-mini .item { display: grid; gap: 8px; align-items: end; background: var(--card-2); border: 1px solid var(--border); border-radius: 8px; padding: 9px; }

/* ---------- Documentación ---------- */
.docs { display: grid; grid-template-columns: 250px minmax(0, 1fr); gap: 26px; max-width: 1220px; }
.docs nav { position: sticky; top: 0; align-self: start; max-height: calc(100vh - 120px); overflow: auto; border-right: 1px solid var(--border); padding-right: 10px; }
.docs nav .titulo { font-size: 11px; letter-spacing: .06em; color: var(--faint); margin: 4px 8px 10px; text-transform: uppercase; }
.docs nav a { display: block; padding: 6px 10px; border-radius: 7px; color: var(--muted); text-decoration: none; font-size: 13px; }
.docs nav a.n1 { color: var(--text); font-weight: 600; }
.docs nav a.n2 { padding-left: 24px; }
.docs nav a:hover { background: var(--card-2); }
.docs article { line-height: 1.6; max-width: 860px; }
.docs article h2 { font-size: 21px; margin: 34px 0 6px; display: flex; gap: 10px; align-items: center; }
.docs article h2:first-child { margin-top: 0; }
.docs article h3 { font-size: 16px; margin: 22px 0 6px; }
.docs article .sub { color: var(--muted); margin: 0 0 14px; }
.docs article p, .docs article li { color: var(--text); }
.docs .tarjetas-doc { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 10px; margin: 12px 0; }
.docs .tarjetas-doc > div { background: var(--card); border: 1px solid var(--border); border-radius: 10px; padding: 12px; font-size: 13px; }
.docs .tarjetas-doc b { display: block; margin-bottom: 4px; }
.docs .pasos { counter-reset: paso; list-style: none; padding: 0; }
.docs .pasos li { counter-increment: paso; position: relative; padding-left: 36px; margin-bottom: 12px; }
.docs .pasos li::before { content: counter(paso); position: absolute; left: 0; top: 1px; width: 24px; height: 24px; border-radius: 50%; background: var(--accent-soft); color: var(--accent); display: grid; place-items: center; font-weight: 700; font-size: 12px; }
.docs .formula { font-family: var(--mono); background: var(--card-2); border: 1px solid var(--border); border-radius: 8px; padding: 10px 12px; font-size: 12.5px; overflow-x: auto; white-space: pre; }
.docs table.tabla { margin: 10px 0; }

/* ---------- Variables de plantilla ---------- */
.variables { max-height: 520px; overflow: auto; }
.variables h4 { font-size: 11.5px; letter-spacing: .05em; text-transform: uppercase; margin: 14px 0 8px; color: var(--muted); }
.variables button { display: block; font-family: var(--mono); font-size: 12px; background: var(--accent-soft); color: var(--accent); border: 0; border-radius: 5px; padding: 3px 7px; cursor: pointer; margin-top: 8px; }
.variables small { display: block; color: var(--faint); font-size: 11.5px; margin-top: 2px; }

@media (max-width: 1250px) {
  .det-grid { grid-template-columns: minmax(0, 1fr) minmax(0, 1fr); }
  .resumen-grid { grid-template-columns: 1fr; }
}
@media (max-width: 1050px) {
  .app { grid-template-columns: 60px 1fr; }
  .app .nav .grupo, .app .nav a span, .app .marca div, .app .pie-sidebar { display: none; }
  .app .nav a { justify-content: center; }
  .form-vehiculo { grid-template-columns: 1fr; }
  .impacto { position: static; }
  .docs { grid-template-columns: 1fr; }
  .docs nav { position: static; max-height: none; border-right: 0; }
}
@media (max-width: 800px) {
  body { overflow: auto; }
  .app { grid-template-columns: 1fr; grid-template-rows: auto 1fr; height: auto; min-height: 100vh; }
  .sidebar { flex-direction: row; overflow-x: auto; }
  .marca { border-bottom: 0; }
  .nav { display: flex; padding: 6px; }
  .pie-sidebar { display: none; }
  #vista { padding: 14px 16px 40px; }
  .grid-2, .grid-3, .grid-4, .kpis, .leyenda-estados, .docs .tarjetas-doc { grid-template-columns: 1fr; }
  #vistaCotizar { min-height: 1400px; }
  .panel-viaje { position: relative; inset: auto; width: auto; margin: 10px; max-height: none; }
  #mapa { position: relative; height: 420px; inset: auto; margin: 0 10px; border-radius: 10px; }
  .panel-detalle { position: fixed; inset: 8px; }
  .det-grid { grid-template-columns: 1fr; }
  .topbar .estado-guardado span.txt { display: none; }
}
@media print {
  .sidebar, .topbar { display: none; }
}

</style>
</head>
<body>
<div class="app" id="app">
  <aside class="sidebar">
    <div class="marca">
      <span id="marcaLogo"></span>
      <div><b id="marcaNombre">Servifácil</b><small>Cotizador de fletes</small></div>
    </div>
    <nav class="nav" id="nav"></nav>
    <div class="pie-sidebar">Creado por INTELIGENCIA DE NEGOCIOS</div>
  </aside>
  <div class="principal">
    <header class="topbar">
      <button class="btn icono fantasma" id="btnColapsar" title="Contraer menú" aria-label="Contraer menú"></button>
      <h1 id="titulo"></h1>
      <div class="espacio"></div>
      <span class="estado-guardado" id="estadoGuardado"><span class="punto"></span><span class="txt">Cargando…</span></span>
      <button class="btn icono fantasma" id="btnTema" title="Cambiar tema claro/oscuro" aria-label="Cambiar tema"></button>
      <button class="usuario-btn" id="btnUsuario" title="Cambiar de usuario"></button>
    </header>
    <main id="vista" tabindex="-1"></main>
    <section id="vistaCotizar" class="oculto">
      <div id="mapa"></div>
      <div class="leyenda-mapa"><span><i></i>Con carga</span><span><i class="v"></i>Sin carga (vacío)</span></div>
      <div class="panel-viaje" id="panelViaje">
        <div class="pv-cab">
          <h2>Nuevo viaje</h2>
          <button class="btn chico suave" id="btnPrimerosPasos"></button>
          <button class="btn icono fantasma" id="btnMinPanel" title="Minimizar panel" aria-label="Minimizar panel"></button>
        </div>
        <div class="pv-cuerpo" id="panelViajeCuerpo"></div>
      </div>
      <div class="panel-detalle oculto" id="panelDetalle"></div>
    </section>
  </div>
</div>
<div id="modales"></div>
<div id="toasts" aria-live="polite"></div>
<input type="file" id="archivoOculto" class="oculto">

<script src="https://cdn.jsdelivr.net/npm/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://unpkg.com/leaflet-image@0.4.0/leaflet-image.js"></script>
<script src="https://cdn.jsdelivr.net/npm/exceljs@4.4.0/dist/exceljs.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/file-saver@2.0.5/dist/FileSaver.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/jspdf@2.5.1/dist/jspdf.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/jspdf-autotable@3.8.4/dist/jspdf.plugin.autotable.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/docx@8.5.0/build/index.umd.js"></script>
<script src="https://cdn.jsdelivr.net/npm/pizzip@3.1.7/dist/pizzip.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/docxtemplater@3.47.4/build/docxtemplater.js"></script>
<script>
// ==MOTOR-INICIO==
/* ============================================================
   MOTOR DE COSTEO
   Funciones puras: reciben el catálogo (datos) y la entrada del
   viaje o renta, y regresan el desglose completo. No tocan el DOM.
   ============================================================ */
var Motor = (function () {
  var PERIODOS = { diario: 365, semanal: 52, quincenal: 24, mensual: 12, bimestral: 6, trimestral: 4, semestral: 2, anual: 1 };

  function n(v) { var x = parseFloat(v); return isFinite(x) ? x : 0; }
  function sum(arr, fn) { return (arr || []).reduce(function (a, x) { return a + n(fn ? fn(x) : x); }, 0); }
  function anualizar(monto, periodo) { return n(monto) * (PERIODOS[periodo] || 1); }

  /* ---------- Tiempo efectivo y tarifa de recuperación ---------- */
  function categoria(datos, catId) {
    var c = (datos.tiempoEfectivo || []).find(function (x) { return x.id === catId; });
    if (!c) return null;
    return { id: c.id, nombre: c.nombre, dias: n(c.dias), horasDia: n(c.horasDia), horasAnio: n(c.dias) * n(c.horasDia), vehiculos: n(c.vehiculos) };
  }

  function tarifaRecuperacion(datos) {
    var gastoAnual = sum(datos.gastosFijos, function (g) { return anualizar(g.monto, g.periodo); });
    var vehiculos = sum(datos.tiempoEfectivo, function (c) { return c.vehiculos; });
    var horas = sum(datos.tiempoEfectivo, function (c) { return n(c.vehiculos) * n(c.dias) * n(c.horasDia); });
    return { gastoAnual: gastoAnual, vehiculos: vehiculos, horasFlota: horas, tarifaHora: horas > 0 ? gastoAnual / horas : 0 };
  }

  /* ---------- Tasas unitarias de un perfil de vehículo ---------- */
  function tasasVehiculo(datos, v, ajustes) {
    ajustes = ajustes || {};
    var cat = categoria(datos, v.categoriaId);
    var horasAnio = cat ? cat.horasAnio : 0;
    var dias = cat ? cat.dias : 0;

    var precioL = ajustes.precioL != null && ajustes.precioL !== '' ? n(ajustes.precioL) : n(v.combustible.precioL);
    var kmLCargado = ajustes.kmLCargado != null && ajustes.kmLCargado !== '' ? n(ajustes.kmLCargado) : n(v.combustible.kmLCargado);
    var kmLVacio = ajustes.kmLVacio != null && ajustes.kmLVacio !== '' ? n(ajustes.kmLVacio) : n(v.combustible.kmLVacio);

    function llantaKm(l) { return n(l.vidaKm) > 0 ? n(l.cantidad) * n(l.precio) / n(l.vidaKm) : 0; }
    var neum = {
      direccion: llantaKm(v.neumaticos.direccionales),
      traccion: llantaKm(v.neumaticos.motrices),
      arrastre: llantaKm(v.neumaticos.arrastre)
    };
    neum.total = neum.direccion + neum.traccion + neum.arrastre;

    var rescate = n(v.valorRescatePct) / 100;
    var activos = [{ nombre: 'Vehículo', valor: n(v.valor), vidaKm: n(v.vidaUtilKm), vidaAnios: n(v.vidaUtilAnios) }]
      .concat((v.activos || []).map(function (a) { return { nombre: a.nombre, valor: n(a.valor), vidaKm: n(a.vidaKm), vidaAnios: n(a.vidaAnios) }; }));
    activos.forEach(function (a) {
      a.depreciable = a.valor * (1 - rescate);
      a.porKm = a.vidaKm > 0 ? a.depreciable / a.vidaKm : 0;
      a.porAnio = a.vidaAnios > 0 ? a.depreciable / a.vidaAnios : 0;
    });
    var depKm = sum(activos, function (a) { return a.porKm; });
    var depAnio = sum(activos, function (a) { return a.porAnio; });

    var mttoKm = n(v.mantenimiento.intervaloKm) > 0 ? n(v.mantenimiento.costo) / n(v.mantenimiento.intervaloKm) : 0;

    var tr = tarifaRecuperacion(datos);
    var gfGenHora = cat ? tr.tarifaHora : 0;
    var gfEspAnual = sum(v.gastosFijos, function (g) { return anualizar(g.monto, g.periodo); });
    var gfEspHora = horasAnio > 0 ? gfEspAnual / horasAnio : 0;

    var equipAnual = sum(v.equipamiento, function (e) {
      var eq = (datos.equipamiento || []).find(function (x) { return x.id === e.equipoId; });
      var vidaAnios = n(e.anios) + n(e.meses) / 12;
      return eq && vidaAnios > 0 ? n(eq.precio) * Math.max(1, n(e.cantidad)) / vidaAnios : 0;
    });
    var equipDia = dias > 0 ? equipAnual / dias : 0;
    var equipHora = horasAnio > 0 ? equipAnual / horasAnio : 0;

    var tasa = n(v.tasaCapitalPct) / 100;
    var valores = sum(activos, function (a) { return a.valor; });
    var capitalPromedio = (valores + valores * rescate) / 2;
    var capitalAnual = capitalPromedio * tasa;
    var capitalHora = horasAnio > 0 ? capitalAnual / horasAnio : 0;

    return {
      categoria: cat, horasAnio: horasAnio, dias: dias,
      precioL: precioL, kmLCargado: kmLCargado, kmLVacio: kmLVacio,
      combKmCargado: kmLCargado > 0 ? precioL / kmLCargado : 0,
      combKmVacio: kmLVacio > 0 ? precioL / kmLVacio : 0,
      ralentiLh: n(v.combustible.ralentiLh), tanqueL: n(v.combustible.tanqueL), recargaMin: n(v.combustible.recargaMin),
      neum: neum, activos: activos, depKm: depKm, depAnio: depAnio, rescatePct: n(v.valorRescatePct),
      mttoKm: mttoKm, mttoIntervalo: n(v.mantenimiento.intervaloKm), mttoCosto: n(v.mantenimiento.costo),
      tarifaRecuperacion: tr, gfGenHora: gfGenHora, gfEspAnual: gfEspAnual, gfEspHora: gfEspHora,
      equipAnual: equipAnual, equipDia: equipDia, equipHora: equipHora,
      tasaCapitalPct: n(v.tasaCapitalPct), capitalPromedio: capitalPromedio, capitalAnual: capitalAnual, capitalHora: capitalHora,
      costoHoraTotal: gfGenHora + gfEspHora + equipHora + capitalHora,
      costoKmCargado: (kmLCargado > 0 ? precioL / kmLCargado : 0) + neum.total + mttoKm + depKm
    };
  }

  var RUBROS_VACIO = [
    { id: 'combustible', nombre: 'Combustible' },
    { id: 'neumaticos', nombre: 'Neumáticos' },
    { id: 'mantenimiento', nombre: 'Mantenimiento' },
    { id: 'depreciacion', nombre: 'Depreciación' },
    { id: 'gfGenerales', nombre: 'Gastos fijos generales' },
    { id: 'gfEspecificos', nombre: 'Gastos fijos específicos' },
    { id: 'equipamiento', nombre: 'Equipamiento' },
    { id: 'capital', nombre: 'Costo de capital' }
  ];

  function vaciosPorDefecto() {
    var o = {};
    RUBROS_VACIO.forEach(function (r) { o[r.id] = { cobrar: true, utilidad: false }; });
    return o;
  }

  /* ---------- Flete por ruta ---------- */
  // entrada.tramos: [{ km, horas, conCarga }]   entrada.paradas: [{ minParada, minRalenti }]
  function costearViaje(datos, v, entrada) {
    var aj = entrada.ajustes || {};
    var t = tasasVehiculo(datos, v, aj);
    var tramos = entrada.tramos || [];

    var kmCargado = sum(tramos.filter(function (x) { return x.conCarga; }), function (x) { return x.km; });
    var kmVacio = sum(tramos.filter(function (x) { return !x.conCarga; }), function (x) { return x.km; });
    var manejoCargadoH = sum(tramos.filter(function (x) { return x.conCarga; }), function (x) { return x.horas; });
    var manejoVacioH = sum(tramos.filter(function (x) { return !x.conCarga; }), function (x) { return x.horas; });
    var manejoH = manejoCargadoH + manejoVacioH;

    var paradasH = sum(entrada.paradas, function (p) { return n(p.minParada) / 60; });
    var ralentiH = sum(entrada.paradas, function (p) { return n(p.minRalenti) / 60; });

    // Descansos obligatorios del operador
    var frec = aj.descansoFrecuenciaH != null ? n(aj.descansoFrecuenciaH) : n(v.descanso.frecuenciaH);
    var durDesc = aj.descansoDuracionH != null ? n(aj.descansoDuracionH) : n(v.descanso.duracionH);
    var costoDesc = aj.costoDescanso != null && aj.costoDescanso !== '' ? n(aj.costoDescanso) : n(v.descanso.costoParada);
    var nDescansos = frec > 0 ? Math.max(0, Math.ceil(manejoH / frec - 1e-9) - 1) : 0;
    var descansoH = nDescansos * durDesc;
    var descansoCosto = nDescansos * costoDesc;

    // Combustible
    var litrosCargado = t.kmLCargado > 0 ? kmCargado / t.kmLCargado : 0;
    var litrosVacio = t.kmLVacio > 0 ? kmVacio / t.kmLVacio : 0;
    var litrosRalenti = ralentiH * t.ralentiLh;
    var litrosTotal = litrosCargado + litrosVacio + litrosRalenti;
    var nRecargas = t.tanqueL > 0 ? Math.max(0, Math.ceil(litrosTotal / t.tanqueL - 1e-9) - 1) : 0;
    var recargaH = nRecargas * t.recargaMin / 60;

    var horasCliente = manejoCargadoH + paradasH + descansoH + recargaH;
    var horasVacio = manejoVacioH;
    var servicioH = manejoH + paradasH + descansoH + recargaH;

    // Columna con utilidad (cliente)
    var c = {
      combustible: (litrosCargado + litrosRalenti) * t.precioL,
      neumaticos: kmCargado * t.neum.total,
      mantenimiento: kmCargado * t.mttoKm,
      depreciacion: kmCargado * t.depKm,
      descanso: descansoCosto,
      gfGenerales: horasCliente * t.gfGenHora,
      gfEspecificos: horasCliente * t.gfEspHora,
      equipamiento: horasCliente * t.equipHora,
      capital: horasCliente * t.capitalHora
    };
    c.subtotal = c.combustible + c.neumaticos + c.mantenimiento + c.depreciacion + c.descanso + c.gfGenerales + c.gfEspecificos + c.equipamiento + c.capital;

    // Tramos vacíos
    var vacioCfg = entrada.vacios || vaciosPorDefecto();
    var vac = {
      combustible: litrosVacio * t.precioL,
      neumaticos: kmVacio * t.neum.total,
      mantenimiento: kmVacio * t.mttoKm,
      depreciacion: kmVacio * t.depKm,
      gfGenerales: horasVacio * t.gfGenHora,
      gfEspecificos: horasVacio * t.gfEspHora,
      equipamiento: horasVacio * t.equipHora,
      capital: horasVacio * t.capitalHora
    };
    var vacioCobrado = 0, vacioConUtilidad = 0, vacioNoCobrado = 0;
    RUBROS_VACIO.forEach(function (r) {
      var cfg = vacioCfg[r.id] || { cobrar: true, utilidad: false };
      if (cfg.cobrar) { vacioCobrado += vac[r.id]; if (cfg.utilidad) vacioConUtilidad += vac[r.id]; }
      else vacioNoCobrado += vac[r.id];
    });

    // Pago del operador
    var p = aj.pago || v.pagoOperador || {};
    var operadores = Math.max(1, Math.round(n(aj.operadores != null ? aj.operadores : (v.pagoOperador && v.pagoOperador.operadores) || 1)));
    var pagoKm = p.porKm ? (n(p.kmCargado) * kmCargado + n(p.kmVacio) * kmVacio) * operadores : 0;
    var pagoHora = p.porHora ? (n(p.horaCargado) * horasCliente + n(p.horaVacio) * horasVacio) * operadores : 0;
    var pagoOperador = pagoKm + pagoHora;
    var pagoConUtilidad = aj.pagoConUtilidad !== false;

    var peajes = n(entrada.peajes);
    var otros = (entrada.otrosGastos || []).map(function (o) { return { concepto: o.concepto, monto: n(o.monto), conUtilidad: !!o.conUtilidad }; });
    var otrosTotal = sum(otros, function (o) { return o.monto; });
    var otrosConUtilidad = sum(otros.filter(function (o) { return o.conUtilidad; }), function (o) { return o.monto; });

    var costoTotalViaje = c.subtotal + vacioCobrado + peajes + pagoOperador;
    var gastoTotal = costoTotalViaje + otrosTotal;

    // Regla comercial interna (opcional, solo clientes del grupo):
    //  - ruta <= límite km: el subtotal se fija en tarifa por m³ de la capacidad del autotanque
    //  - ruta > límite km: se fuerza el % de utilidad indicado
    var regla = entrada.reglaInterna || null;
    var reglaAplicada = null;
    var utilidadPct = n(entrada.utilidadPct);
    if (regla && regla.activa && n(v.capacidadLitros) > 0) {
      if (kmCargado + kmVacio <= n(regla.limiteKm)) {
        var m3 = n(v.capacidadLitros) / 1000;
        reglaAplicada = { tipo: 'tarifaFija', tarifaM3: n(regla.tarifaM3), m3: m3, subtotal: n(regla.tarifaM3) * m3, limiteKm: n(regla.limiteKm) };
      } else {
        utilidadPct = n(regla.utilidadRutaLarga);
        reglaAplicada = { tipo: 'utilidadFija', utilidadPct: utilidadPct, limiteKm: n(regla.limiteKm) };
      }
    }
    var baseUtilidad = c.subtotal + vacioConUtilidad + (pagoConUtilidad ? pagoOperador : 0) + otrosConUtilidad;
    var utilidad = baseUtilidad * utilidadPct / 100;
    var financiamiento = gastoTotal * n(entrada.financiamientoPct) / 100;
    var aumentoCarga = gastoTotal * n(entrada.aumentoPct) / 100;
    var subtotalCliente = gastoTotal + utilidad + financiamiento + aumentoCarga;
    if (reglaAplicada && reglaAplicada.tipo === 'tarifaFija') {
      subtotalCliente = reglaAplicada.subtotal;
      utilidad = subtotalCliente - gastoTotal - financiamiento - aumentoCarga;
    }

    var comisionNeto = p.comisionNeto ? subtotalCliente * n(p.pctNeto) / 100 : 0;
    var comisionUtilidad = p.comisionUtilidad ? Math.max(0, utilidad) * n(p.pctUtilidad) / 100 : 0;
    var ivaPct = n(entrada.ivaPct);
    var iva = subtotalCliente * ivaPct / 100;
    var totalConIva = subtotalCliente + iva;
    var kmTotal = kmCargado + kmVacio;
    var toneladas = n(entrada.toneladas);
    var litros = n(v.capacidadLitros);

    return {
      tasas: t, operadores: operadores,
      km: { cargado: kmCargado, vacio: kmVacio, total: kmTotal },
      tiempo: { manejoH: manejoH, manejoCargadoH: manejoCargadoH, manejoVacioH: manejoVacioH, paradasH: paradasH, descansoH: descansoH, recargaH: recargaH, ralentiH: ralentiH, servicioH: servicioH, horasCliente: horasCliente, horasVacio: horasVacio },
      descansos: { n: nDescansos, frecuenciaH: frec, duracionH: durDesc, costoParada: costoDesc, costo: descansoCosto },
      combustibleDet: { litrosCargado: litrosCargado, litrosVacio: litrosVacio, litrosRalenti: litrosRalenti, litrosTotal: litrosTotal, costoRalenti: litrosRalenti * t.precioL, costoTotal: litrosTotal * t.precioL, recargas: nRecargas },
      cliente: c,
      vacio: vac, vacioCobrado: vacioCobrado, vacioConUtilidad: vacioConUtilidad, vacioNoCobrado: vacioNoCobrado, vacioCfg: vacioCfg,
      pago: { porKm: pagoKm, porHora: pagoHora, total: pagoOperador, conUtilidad: pagoConUtilidad, comisionNeto: comisionNeto, comisionUtilidad: comisionUtilidad },
      peajes: peajes, otros: otros, otrosTotal: otrosTotal,
      costoTotalViaje: costoTotalViaje, gastoTotal: gastoTotal,
      utilidadPct: utilidadPct, baseUtilidad: baseUtilidad, utilidad: utilidad,
      utilidadNeta: utilidad - comisionNeto - comisionUtilidad,
      financiamientoPct: n(entrada.financiamientoPct), financiamiento: financiamiento,
      aumentoPct: n(entrada.aumentoPct), aumentoCarga: aumentoCarga,
      reglaAplicada: reglaAplicada,
      subtotalCliente: subtotalCliente, ivaPct: ivaPct, iva: iva, totalConIva: totalConIva,
      porKm: kmTotal > 0 ? totalConIva / kmTotal : 0,
      porTonelada: toneladas > 0 ? totalConIva / toneladas : 0,
      porLitro: litros > 0 ? subtotalCliente / litros : 0,
      porM3: litros > 0 ? subtotalCliente / (litros / 1000) : 0,
      margenSobreCosto: gastoTotal > 0 ? (subtotalCliente - gastoTotal) / gastoTotal : 0
    };
  }

  /* ---------- Renta / disposición ---------- */
  function horasPeriodo(cat, unidad, cantidad) {
    var q = Math.max(0, n(cantidad));
    if (!cat) return 0;
    switch (unidad) {
      case 'hora': return q;
      case 'jornada': case 'dia': return q * cat.horasDia;
      case 'semana': return q * cat.horasAnio / 52;
      case 'mensual': case 'fija': return q * cat.horasAnio / 12;
      default: return 0;
    }
  }

  function costearRenta(datos, v, r) {
    var t = tasasVehiculo(datos, v, r.ajustes || {});
    var cat = t.categoria;
    var horas = horasPeriodo(cat, r.unidad, r.cantidad);
    var fraccion = t.horasAnio > 0 ? horas / t.horasAnio : 0;
    var meses = fraccion * 12;
    var operadores = Math.max(1, Math.round(n(r.operadores) || 1));

    var disp = {
      gfGenerales: horas * t.gfGenHora,
      gfEspecificos: horas * t.gfEspHora,
      equipamiento: horas * t.equipHora,
      depreciacion: t.depAnio * fraccion,
      capital: horas * t.capitalHora,
      sueldo: n(r.sueldoOperadorMensual) * operadores * meses
    };
    disp.total = disp.gfGenerales + disp.gfEspecificos + disp.equipamiento + disp.depreciacion + disp.capital + disp.sueldo;

    var km = n(r.kmPeriodo);
    var usoKm = (r.combustibleIncluido ? t.combKmCargado : 0) + t.neum.total + t.mttoKm;
    var uso = {
      combustible: r.combustibleIncluido ? km * t.combKmCargado : 0,
      neumaticos: km * t.neum.total,
      mantenimiento: km * t.mttoKm,
      casetas: r.casetasIncluidas ? n(r.casetasPeriodo) : 0
    };
    uso.total = uso.combustible + uso.neumaticos + uso.mantenimiento + uso.casetas;

    var costoBase = disp.total + uso.total;
    var utilidadPct = n(r.utilidadPct);
    var utilidad = costoBase * utilidadPct / 100;
    var subtotal = costoBase + utilidad;
    var precioSugerido = subtotal;
    var margen = null;
    if (r.unidad === 'fija') {
      var mesesFija = Math.max(1, n(r.cantidad));
      var precioMes = r.precioFijoMensual != null && r.precioFijoMensual !== '' ? n(r.precioFijoMensual) : subtotal / mesesFija;
      subtotal = precioMes * mesesFija;
      utilidad = subtotal - costoBase;
      margen = subtotal > 0 ? utilidad / subtotal : 0;
    }
    var recargo = r.recargoExcedentePct != null ? n(r.recargoExcedentePct) : 50;
    var tope = n(r.topeKm);
    var excedenteKm = tope > 0 && km > tope ? km - tope : 0;
    var tarifaKmExcedente = usoKm * (1 + recargo / 100) * (1 + utilidadPct / 100);
    var tarifaHoraExcedente = horas > 0 ? (disp.total / horas) * (1 + recargo / 100) * (1 + utilidadPct / 100) : 0;
    var ivaPct = n(r.ivaPct);
    var iva = subtotal * ivaPct / 100;

    return {
      tasas: t, horas: horas, fraccionAnio: fraccion, meses: meses, operadores: operadores,
      disponibilidad: disp, uso: uso, usoPorKm: usoKm, km: km, costoBase: costoBase,
      utilidadPct: utilidadPct, utilidad: utilidad, precioSugerido: precioSugerido,
      precioSugeridoMensual: r.unidad === 'fija' ? precioSugerido / Math.max(1, n(r.cantidad)) : null,
      margen: margen, subtotal: subtotal, ivaPct: ivaPct, iva: iva, total: subtotal + iva,
      tope: { km: tope, excedenteKm: excedenteKm, recargoPct: recargo, tarifaKm: tarifaKmExcedente, tarifaHora: tarifaHoraExcedente, montoExcedente: excedenteKm * tarifaKmExcedente }
    };
  }

  function redondear(x, d) { var f = Math.pow(10, d == null ? 2 : d); return Math.round((n(x) + Number.EPSILON) * f) / f; }

  return {
    PERIODOS: PERIODOS, RUBROS_VACIO: RUBROS_VACIO, anualizar: anualizar, categoria: categoria,
    tarifaRecuperacion: tarifaRecuperacion, tasasVehiculo: tasasVehiculo, vaciosPorDefecto: vaciosPorDefecto,
    costearViaje: costearViaje, horasPeriodo: horasPeriodo, costearRenta: costearRenta, redondear: redondear
  };
})();
// ==MOTOR-FIN==

// ==SEMILLA-INICIO==
/* ============================================================
   DATOS INICIALES (EJEMPLO)
   Reproducen el ejemplo del manual de referencia (tractocamión
   T3-S3, ruta Pajaritos–Sayula). Son valores ilustrativos: deben
   sustituirse por los datos reales de la operación.
   ============================================================ */
function datosIniciales() {
  var hoy = new Date().toISOString();
  function gf(id, nombre, periodo, monto, descripcion) { return { id: id, nombre: nombre, periodo: periodo, monto: monto, descripcion: descripcion, creado: hoy }; }
  function eq(id, equipo, descripcion, precio) { return { id: id, equipo: equipo, descripcion: descripcion, precio: precio, creado: hoy }; }

  var equipamiento = [
    eq('eq01', 'Lona / cubierta para carga', 'Amarre y sujeción de carga', 1500),
    eq('eq02', 'Esquineros / protectores de carga (juego)', 'Amarre y sujeción de carga', 200),
    eq('eq03', 'Cuerda / soga de nylon (rollo)', 'Amarre y sujeción de carga', 400),
    eq('eq04', 'Pulpo de acero 1/2" x 10 ft con gancho', 'Amarre y sujeción de carga', 1590),
    eq('eq05', 'Eslinga sintética 3" x 3 m', 'Amarre y sujeción de carga', 320),
    eq('eq06', 'Banda de amarre con matraca 4" x 9 m', 'Amarre y sujeción de carga', 1200),
    eq('eq07', 'Banda de amarre con matraca 2" x 9 m', 'Amarre y sujeción de carga', 400),
    eq('eq08', 'Tensor / templador para cadena (binder)', 'Amarre y sujeción de carga', 350),
    eq('eq09', 'Cadena grado 70 con ganchos (3/8" x 6 m)', 'Amarre y sujeción de carga', 650),
    eq('eq10', 'Maneral y dados para birlos', 'Herramienta del camión', 450),
    eq('eq11', 'Llave de cruz para birlos', 'Herramienta del camión', 350),
    eq('eq12', 'Caja de herramienta básica', 'Herramienta del camión', 1200),
    eq('eq13', 'Lámpara de emergencia / baliza', 'Equipo de seguridad y carretera', 300),
    eq('eq14', 'Cables pasacorriente', 'Equipo de seguridad y carretera', 600),
    eq('eq15', 'Cuñas / tacos de bloqueo (par)', 'Equipo de seguridad y carretera', 400),
    eq('eq16', 'Gato hidráulico de botella', 'Equipo de seguridad y carretera', 1500)
  ];

  var gastosFijos = [
    gf('gf01', 'Combustible de vehículos administrativos', 'mensual', 7000, 'Combustible de autos administrativos.'),
    gf('gf02', 'Equipo de cómputo y software', 'mensual', 4000, 'Equipos y licencias de oficina.'),
    gf('gf03', 'Capacitación y licencias federales', 'mensual', 5000, 'Cursos y licencias federales del personal.'),
    gf('gf04', 'Publicidad, sitio web y marketing', 'mensual', 6000, 'Promoción y presencia digital.'),
    gf('gf05', 'Comisiones y servicios bancarios', 'mensual', 2500, 'Comisiones de cuentas y terminales.'),
    gf('gf06', 'Papelería y consumibles de oficina', 'mensual', 3500, 'Material de oficina.'),
    gf('gf07', 'Asesoría legal', 'mensual', 8000, 'Asesoría jurídica.'),
    gf('gf08', 'Despacho / honorarios contables', 'mensual', 18000, 'Contabilidad externa.'),
    gf('gf09', 'Membresía CANACAR', 'anual', 24000, 'Cuota de la cámara del autotransporte.'),
    gf('gf10', 'Permiso federal SICT (autotransporte)', 'anual', 18000, 'Permiso/trámites de autotransporte federal.'),
    gf('gf11', 'Software de facturación (CFDI)', 'mensual', 3500, 'Sistema de facturación y timbrado.'),
    gf('gf12', 'Plataforma de GPS/telemetría', 'mensual', 12000, 'Suscripción de la plataforma (no incluye equipos).'),
    gf('gf13', 'Sueldos base de operadores', 'mensual', 240000, '30 operadores × $4,000 quincenales.'),
    gf('gf14', 'IMSS, INFONAVIT y cargas sociales', 'mensual', 110000, 'Cuotas patronales IMSS e INFONAVIT.'),
    gf('gf15', 'Vigilancia del patio (velador)', 'mensual', 28000, 'Personal de seguridad contratado.'),
    gf('gf16', 'Mecánico de planta', 'mensual', 14000, 'Mecánico contratado para el mantenimiento menor.'),
    gf('gf17', 'Personal de patio', 'mensual', 12000, 'Personal contratado para maniobras.'),
    gf('gf18', 'Coordinación de tráfico / despacho', 'mensual', 30000, 'Coordinadores que asignan y siguen viajes.'),
    gf('gf19', 'Administrativos (contabilidad, compras)', 'mensual', 72000, 'Nómina del personal administrativo.'),
    gf('gf20', 'Dirección / gerencia', 'mensual', 55000, 'Sueldo de dirección general / gerencia.'),
    gf('gf21', 'Impuesto predial', 'anual', 36000, 'Predial del inmueble (pago anual).'),
    gf('gf22', 'Limpieza y mantenimiento de instalaciones', 'mensual', 9000, 'Aseo y conservación de oficinas y patio.'),
    gf('gf23', 'Internet y telefonía', 'mensual', 4500, 'Enlaces de datos y líneas telefónicas.'),
    gf('gf24', 'Agua', 'mensual', 2500, 'Servicio de agua.'),
    gf('gf25', 'Energía eléctrica (CFE)', 'mensual', 14000, 'Consumo de oficinas y patio.'),
    gf('gf26', 'Renta de oficinas y patio', 'mensual', 50000, 'Inmueble administrativo y patio de maniobras.')
  ];

  var tiempoEfectivo = [
    { id: 'grandes', nombre: 'Grandes', vehiculos: 10, dias: 312, horasDia: 8 },
    { id: 'medianos', nombre: 'Medianos', vehiculos: 20, dias: 312, horasDia: 8 },
    { id: 'pequenos', nombre: 'Pequeños', vehiculos: 10, dias: 312, horasDia: 8 }
  ];

  function perfilBase(id, nombre, descripcion, tipo, capacidadLitros, ejemplo) {
    return {
      id: id, nombre: nombre, descripcion: descripcion, tipo: tipo, categoriaId: 'grandes', ejemplo: ejemplo, creado: hoy,
      dimensiones: { alto: 4.1, ancho: 2.6, largo: 22, peso: 15, capRemolque: 30 },
      capacidadLitros: capacidadLitros,
      valor: 1373447, vidaUtilKm: 700000, vidaUtilAnios: 10, tasaCapitalPct: 10, valorRescatePct: 20,
      neumaticos: {
        direccionales: { cantidad: 2, vidaKm: 120000, precio: 3345 },
        motrices: { cantidad: 8, vidaKm: 150000, precio: 3345 },
        arrastre: { cantidad: 12, vidaKm: 120000, precio: 3345 }
      },
      combustible: { kmLVacio: 2.1, kmLCargado: 1.9, precioL: 23.41, tanqueL: 800, recargaMin: 20, ralentiLh: 3 },
      descanso: { frecuenciaH: 8, duracionH: 8, costoParada: 800 },
      mantenimiento: { intervaloKm: 25000, costo: 10000 },
      activos: [{ id: 'ac1', nombre: 'Semirremolque', valor: 550000, vidaKm: 1000000, vidaAnios: 15 }],
      gastosFijos: [
        { id: 'gv1', nombre: 'Seguro de la unidad', monto: 72000, periodo: 'anual' },
        { id: 'gv2', nombre: 'GPS / telemetría de la unidad', monto: 1100, periodo: 'mensual' },
        { id: 'gv3', nombre: 'Placas, tenencia, verificaciones y permisos', monto: 15014, periodo: 'anual' }
      ],
      equipamiento: [
        { id: 've01', equipoId: 'eq06', cantidad: 4, anios: 1, meses: 0 },
        { id: 've02', equipoId: 'eq12', cantidad: 1, anios: 3, meses: 0 },
        { id: 've03', equipoId: 'eq16', cantidad: 1, anios: 2, meses: 0 },
        { id: 've04', equipoId: 'eq13', cantidad: 3, anios: 1, meses: 0 },
        { id: 've05', equipoId: 'eq14', cantidad: 1, anios: 2, meses: 0 },
        { id: 've06', equipoId: 'eq15', cantidad: 2, anios: 1, meses: 0 },
        { id: 've07', equipoId: 'eq11', cantidad: 1, anios: 2, meses: 0 },
        { id: 've08', equipoId: 'eq10', cantidad: 1, anios: 2, meses: 0 },
        { id: 've09', equipoId: 'eq02', cantidad: 4, anios: 0, meses: 6 },
        { id: 've10', equipoId: 'eq09', cantidad: 2, anios: 2, meses: 0 },
        { id: 've11', equipoId: 'eq08', cantidad: 2, anios: 2, meses: 0 },
        { id: 've12', equipoId: 'eq05', cantidad: 2, anios: 1, meses: 0 },
        { id: 've13', equipoId: 'eq01', cantidad: 2, anios: 1, meses: 0 },
        { id: 've14', equipoId: 'eq04', cantidad: 1, anios: 1, meses: 0 },
        { id: 've15', equipoId: 'eq03', cantidad: 1, anios: 0, meses: 9 },
        { id: 've16', equipoId: 'eq07', cantidad: 3, anios: 2, meses: 8 }
      ],
      pagoOperador: { operadores: 1, porKm: true, kmCargado: 3, kmVacio: 2, porHora: false, horaCargado: 0, horaVacio: 0, comisionNeto: false, pctNeto: 0, comisionUtilidad: false, pctUtilidad: 0 }
    };
  }

  var vehiculos = [
    perfilBase('v-t3s3', 'Tractocamión caja seca (T3-S3)', 'Tractocamión articulado 6 ejes 22 llantas · ejemplo del manual', 'T3-S3', 0, true),
    perfilBase('v-at20', 'Autotanque 20,000 L', 'Autotanque para petrolíferos · capturar datos reales', 'T3-S2', 20000, true),
    perfilBase('v-at30', 'Autotanque 30,000 L', 'Autotanque para petrolíferos · capturar datos reales', 'T3-S2', 30000, true),
    perfilBase('v-at40', 'Autotanque 40,000 L', 'Autotanque para petrolíferos · capturar datos reales', 'T3-S3', 40000, true),
    perfilBase('v-at60', 'Full 60,000 L', 'Doble autotanque · capturar datos reales', 'T3-S2-R4', 60000, true)
  ];

  return {
    version: 1,
    empresa: {
      nombre: 'Servifácil', legalName: '', rfc: '', direccion: '', telefono: '', email: '', website: '', logo: ''
    },
    folios: { prefijo: 'COT', incluirAnio: true, reinicioAnual: true, consecutivo: 0, anio: new Date().getFullYear() },
    parametros: {
      ivaPct: 16, utilidadPct: 20, financiamientoPct: 0, vigenciaDias: 30,
      factorTrafico: 1.15, velMaxDefault: 90,
      ruteo: { proveedor: 'osrm', orsApiKey: '' },
      directorioCsv: 'DISTANCIA_ENTRE_TAD_ESTACION (1).csv',
      reglaGrupo: { activa: true, tarifaM3: 170.77, limiteKm: 100, utilidadRutaLarga: 20 }
    },
    tiempoEfectivo: tiempoEfectivo,
    gastosFijos: gastosFijos,
    equipamiento: equipamiento,
    vehiculos: vehiculos,
    clientes: [],
    tiposCarga: [
      { id: 'general', nombre: 'Carga general', aumentoPct: 0 },
      { id: 'peligrosa', nombre: 'Material peligroso (petrolíferos)', aumentoPct: 0 }
    ],
    casetas: [],
    zonasRestringidas: [
      { id: 'z1', nombre: 'Poblado de Chiltepec', lat: 18.43156, lon: -93.0848, radioKm: 3.0,
        alternas: [
          { nombre: 'Paraíso - Comalcalco - Cárdenas - Villahermosa', puntos: [[18.3975, -93.2135], [18.2632, -93.2234], [17.9897, -93.3756], [17.9895, -92.9475]] },
          { nombre: 'Villahermosa - Cárdenas - Comalcalco - Paraíso', puntos: [[17.9895, -92.9475], [17.9897, -93.3756], [18.2632, -93.2234], [18.3975, -93.2135]] }
        ] }
    ],
    rutas: [],
    cotizaciones: [],
    tarifarios: [],
    plantillas: [],
    plantillaDefault: { cotizacion: 'default', precios: 'default' },
    roles: [
      { id: 'admin', nombre: 'admin', sistema: true, permisos: ['*'] },
      { id: 'usuario', nombre: 'usuario', sistema: true, permisos: ['inicio', 'cotizar', 'cotizaciones', 'precios', 'clientes', 'documentacion'] }
    ],
    usuarios: [
      { id: 'u1', nombre: 'Administrador', email: '', puesto: '', rolId: 'admin', estatus: 'alta' }
    ],
    usuarioActivoId: 'u1'
  };
}
// ==SEMILLA-FIN==

'use strict';
/* ============================================================
   NÚCLEO: utilidades, íconos, estado, persistencia y navegación
   ============================================================ */
var $ = function (s, r) { return (r || document).querySelector(s); };
var $$ = function (s, r) { return Array.prototype.slice.call((r || document).querySelectorAll(s)); };

var ICONOS = {
  home: '<path d="M3 10.5 12 3l9 7.5V21a1 1 0 0 1-1 1h-5v-7h-6v7H4a1 1 0 0 1-1-1z"/>',
  globe: '<circle cx="12" cy="12" r="9"/><path d="M3 12h18M12 3a14 14 0 0 1 0 18M12 3a14 14 0 0 0 0 18"/>',
  fileText: '<path d="M14 3H6a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V9z"/><path d="M14 3v6h6M8 13h8M8 17h6"/>',
  truck: '<path d="M3 6h11v10H3zM14 9h4l3 3v4h-7"/><circle cx="7" cy="18" r="2"/><circle cx="17" cy="18" r="2"/>',
  tool: '<path d="M14.7 6.3a4 4 0 0 0-5.4 5.4L3 18l3 3 6.3-6.3a4 4 0 0 0 5.4-5.4l-2.5 2.5-2.4-.6-.6-2.4z"/>',
  dollar: '<circle cx="12" cy="12" r="9"/><path d="M15 9.5c-.5-1-1.6-1.5-3-1.5-1.7 0-3 .8-3 2s1.3 1.8 3 2 3 .8 3 2-1.3 2-3 2c-1.4 0-2.5-.5-3-1.5M12 6v2M12 16v2"/>',
  users: '<circle cx="9" cy="8" r="3.5"/><path d="M2.5 20a6.5 6.5 0 0 1 13 0M16 4.5a3.5 3.5 0 0 1 0 7M18 14a6.5 6.5 0 0 1 3.5 6"/>',
  user: '<circle cx="12" cy="8" r="4"/><path d="M4 21a8 8 0 0 1 16 0"/>',
  clipboard: '<rect x="5" y="4" width="14" height="17" rx="2"/><path d="M9 4V3h6v1M9 10h6M9 14h6M9 18h4"/>',
  file: '<path d="M14 3H6a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V9z"/><path d="M14 3v6h6"/>',
  settings: '<circle cx="12" cy="12" r="3"/><path d="M12 2v3M12 19v3M4.2 4.2l2.1 2.1M17.7 17.7l2.1 2.1M2 12h3M19 12h3M4.2 19.8l2.1-2.1M17.7 6.3l2.1-2.1"/>',
  book: '<path d="M4 4.5A2.5 2.5 0 0 1 6.5 2H20v17H6.5A2.5 2.5 0 0 0 4 21.5zM4 21.5A2.5 2.5 0 0 1 6.5 19H20v3H6.5"/>',
  shield: '<path d="M12 3 4 6v6c0 5 3.4 8.4 8 9 4.6-.6 8-4 8-9V6z"/>',
  plus: '<path d="M12 5v14M5 12h14"/>',
  minus: '<path d="M5 12h14"/>',
  upload: '<path d="M12 15V3M7 8l5-5 5 5M4 15v4a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-4"/>',
  download: '<path d="M12 3v12M7 10l5 5 5-5M4 15v4a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-4"/>',
  filter: '<path d="M3 5h18l-7 8v6l-4 2v-8z"/>',
  search: '<circle cx="11" cy="11" r="7"/><path d="m20 20-3.5-3.5"/>',
  x: '<path d="M6 6l12 12M18 6 6 18"/>',
  trash: '<path d="M4 7h16M9 7V4h6v3M6 7l1 13a1 1 0 0 0 1 1h8a1 1 0 0 0 1-1l1-13"/>',
  edit: '<path d="M4 20h4L19 9l-4-4L4 16zM14 6l4 4"/>',
  copy: '<rect x="8" y="8" width="12" height="12" rx="2"/><path d="M16 8V6a2 2 0 0 0-2-2H6a2 2 0 0 0-2 2v8a2 2 0 0 0 2 2h2"/>',
  refresh: '<path d="M20 11a8 8 0 0 0-14.6-4.5L4 8M4 4v4h4M4 13a8 8 0 0 0 14.6 4.5L20 16M20 20v-4h-4"/>',
  eye: '<path d="M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z"/><circle cx="12" cy="12" r="3"/>',
  check: '<path d="m5 12 5 5L20 7"/>',
  checkCircle: '<circle cx="12" cy="12" r="9"/><path d="m8 12 3 3 5-6"/>',
  xCircle: '<circle cx="12" cy="12" r="9"/><path d="m9 9 6 6M15 9l-6 6"/>',
  alert: '<path d="M12 3 2 21h20zM12 10v5M12 18v.01"/>',
  clock: '<circle cx="12" cy="12" r="9"/><path d="M12 7v5l3 2"/>',
  route: '<circle cx="6" cy="19" r="2.5"/><circle cx="18" cy="5" r="2.5"/><path d="M8.5 19H16a3.5 3.5 0 0 0 0-7H8a3.5 3.5 0 0 1 0-7h7.5"/>',
  save: '<path d="M5 3h11l3 3v13a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2z"/><path d="M7 3v5h8V3M7 21v-7h10v7"/>',
  sigma: '<path d="M18 5H6l6 7-6 7h12"/>',
  chevR: '<path d="m9 6 6 6-6 6"/>',
  chevD: '<path d="m6 9 6 6 6-6"/>',
  chevU: '<path d="m6 15 6-6 6 6"/>',
  eraser: '<path d="m7 21-4-4L14 6l6 6-9 9zM14 21h7M9.5 10.5l6 6"/>',
  bookmark: '<path d="M6 3h12v18l-6-4-6 4z"/>',
  pin: '<path d="M12 21s-7-6.3-7-12a7 7 0 0 1 14 0c0 5.7-7 12-7 12z"/><circle cx="12" cy="9" r="2.5"/>',
  undo: '<path d="M9 14 4 9l5-5"/><path d="M4 9h10.5a5.5 5.5 0 0 1 0 11H11"/>',
  info: '<circle cx="12" cy="12" r="9"/><path d="M12 11v5M12 8v.01"/>',
  help: '<circle cx="12" cy="12" r="9"/><path d="M9.5 9.5a2.5 2.5 0 0 1 4.9.7c0 1.7-2.4 2.2-2.4 3.8M12 17v.01"/>',
  lock: '<rect x="5" y="11" width="14" height="10" rx="2"/><path d="M8 11V7a4 4 0 0 1 8 0v4"/>',
  grip: '<circle cx="9" cy="6" r="1"/><circle cx="15" cy="6" r="1"/><circle cx="9" cy="12" r="1"/><circle cx="15" cy="12" r="1"/><circle cx="9" cy="18" r="1"/><circle cx="15" cy="18" r="1"/>',
  dots: '<circle cx="12" cy="5" r="1"/><circle cx="12" cy="12" r="1"/><circle cx="12" cy="19" r="1"/>',
  calendar: '<rect x="3" y="5" width="18" height="16" rx="2"/><path d="M3 10h18M8 3v4M16 3v4"/>',
  fuel: '<path d="M4 21V5a2 2 0 0 1 2-2h7a2 2 0 0 1 2 2v16M3 21h13M4 10h11M15 8l3 3v6a1.5 1.5 0 0 0 3 0V9l-3-3"/>',
  chart: '<path d="M4 20V10M10 20V4M16 20v-8M22 20H2"/>',
  bed: '<path d="M3 18V6M3 13h18v5M21 13a3 3 0 0 0-3-3h-7v3"/><circle cx="7" cy="10" r="1.5"/>',
  trendDown: '<path d="m3 7 7 7 4-4 7 7M21 11v6h-6"/>',
  bank: '<path d="M3 10 12 4l9 6M5 10v8M9.5 10v8M14.5 10v8M19 10v8M3 21h18"/>',
  tire: '<circle cx="12" cy="12" r="9"/><circle cx="12" cy="12" r="4"/>',
  box: '<path d="M3 7.5 12 3l9 4.5v9L12 21l-9-4.5z"/><path d="M3 7.5 12 12l9-4.5M12 12v9"/>',
  moon: '<path d="M20 14.5A8 8 0 1 1 9.5 4a6.5 6.5 0 0 0 10.5 10.5z"/>',
  sun: '<circle cx="12" cy="12" r="4"/><path d="M12 2v2M12 20v2M4.9 4.9l1.4 1.4M17.7 17.7l1.4 1.4M2 12h2M20 12h2M4.9 19.1l1.4-1.4M17.7 6.3l1.4-1.4"/>',
  panel: '<rect x="3" y="4" width="18" height="16" rx="2"/><path d="M9 4v16"/>',
  table: '<rect x="3" y="4" width="18" height="16" rx="2"/><path d="M3 10h18M3 15h18M9 4v16"/>',
  star: '<path d="m12 3 2.8 5.8 6.2.9-4.5 4.4 1 6.2L12 17.3 6.5 20.3l1-6.2L3 9.7l6.2-.9z"/>',
  target: '<circle cx="12" cy="12" r="9"/><circle cx="12" cy="12" r="5"/><circle cx="12" cy="12" r="1"/>',
  hash: '<path d="M5 9h14M5 15h14M10 3 8 21M16 3l-2 18"/>',
  bulb: '<path d="M9 18h6M10 21h4M12 3a6 6 0 0 0-4 10.5c.7.7 1 1.5 1 2.5h6c0-1 .3-1.8 1-2.5A6 6 0 0 0 12 3z"/>',
  database: '<ellipse cx="12" cy="5" rx="8" ry="3"/><path d="M4 5v14c0 1.7 3.6 3 8 3s8-1.3 8-3V5M4 12c0 1.7 3.6 3 8 3s8-1.3 8-3"/>',
  split: '<path d="M6 3v5a6 6 0 0 0 6 6M18 3v5a6 6 0 0 1-6 6v7"/>',
  map: '<path d="M9 4 3 6v14l6-2 6 2 6-2V4l-6 2zM9 4v14M15 6v14"/>',
  send: '<path d="M22 2 11 13M22 2l-7 20-4-9-9-4z"/>',
  hourglass: '<path d="M6 3h12M6 21h12M7 3c0 5 10 5 10 9s-10 4-10 9M17 3c0 5-10 5-10 9s10 4 10 9"/>',
  gauge: '<path d="M12 14l4-4M3.5 17a9 9 0 1 1 17 0"/>',
  minimize: '<path d="M4 14h6v6M20 10h-6V4M14 10l7-7M3 21l7-7"/>',
  maximize: '<path d="M15 3h6v6M9 21H3v-6M21 3l-7 7M3 21l7-7"/>',
  word: '<path d="M14 3H6a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V9z"/><path d="M14 3v6h6M8 12l1.5 6 2.5-5 2.5 5L16 12"/>',
  pdf: '<path d="M14 3H6a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V9z"/><path d="M14 3v6h6M8 17v-4h1.5a1.5 1.5 0 0 1 0 3H8"/>'
};
function ico(nombre, extra) { return '<svg class="i ' + (extra || '') + '" viewBox="0 0 24 24" aria-hidden="true">' + (ICONOS[nombre] || '') + '</svg>'; }

function uid(p) { return (p || 'id') + '-' + Math.random().toString(36).slice(2, 8) + Date.now().toString(36).slice(-4); }
function esc(s) { return String(s == null ? '' : s).replace(/[&<>"']/g, function (c) { return { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c]; }); }
function num(v) { var x = parseFloat(v); return isFinite(x) ? x : 0; }
function clonar(o) { return JSON.parse(JSON.stringify(o)); }
function getPath(o, ruta) { return ruta.split('.').reduce(function (a, k) { return a == null ? undefined : a[k]; }, o); }
function setPath(o, ruta, v) { var ks = ruta.split('.'); var u = ks.pop(); var t = ks.reduce(function (a, k) { if (a[k] == null) a[k] = {}; return a[k]; }, o); t[u] = v; }
function debounce(fn, ms) { var t; return function () { var a = arguments, s = this; clearTimeout(t); t = setTimeout(function () { fn.apply(s, a); }, ms); }; }

var fmt = {
  mon: function (v, d) { d = d == null ? 2 : d; return '$' + num(v).toLocaleString('es-MX', { minimumFractionDigits: d, maximumFractionDigits: d }); },
  num: function (v, d) { d = d == null ? 2 : d; return num(v).toLocaleString('es-MX', { minimumFractionDigits: d, maximumFractionDigits: d }); },
  ent: function (v) { return Math.round(num(v)).toLocaleString('es-MX'); },
  pct: function (v, d) { return fmt.num(v, d == null ? 1 : d) + '%'; },
  km: function (v, d) { return fmt.num(v, d == null ? 1 : d) + ' km'; },
  dur: function (h) {
    var m = Math.round(num(h) * 60); if (m < 60) return m + ' min';
    var hh = Math.floor(m / 60), mm = m % 60; return hh + ' h' + (mm ? ' ' + mm + ' min' : '');
  },
  hm: function (h) { var m = Math.round(num(h) * 60); return Math.floor(m / 60) + 'h ' + (m % 60) + 'min.'; },
  fecha: function (iso) { if (!iso) return ''; var d = new Date(iso.length <= 10 ? iso + 'T12:00:00' : iso); return isNaN(d) ? '' : d.toLocaleDateString('es-MX', { day: '2-digit', month: '2-digit', year: 'numeric' }); },
  fechaHora: function (iso) { if (!iso) return ''; var d = new Date(iso); return isNaN(d) ? '' : d.toLocaleDateString('es-MX', { day: '2-digit', month: '2-digit', year: 'numeric' }) + ', ' + d.toLocaleTimeString('es-MX', { hour: '2-digit', minute: '2-digit' }); },
  fechaLarga: function (iso) { if (!iso) return ''; var d = new Date(iso.length <= 10 ? iso + 'T12:00:00' : iso); return d.toLocaleDateString('es-MX', { day: 'numeric', month: 'long', year: 'numeric' }); }
};
function hoyISO() { var d = new Date(); return new Date(d.getTime() - d.getTimezoneOffset() * 60000).toISOString().slice(0, 10); }
function sumarDias(iso, dias) { var d = new Date(iso + 'T12:00:00'); d.setDate(d.getDate() + dias); return d.toISOString().slice(0, 10); }

/* ---------- Avisos emergentes, modales y confirmaciones ---------- */
function toast(msg, tipo, ms) {
  var el = document.createElement('div');
  el.className = 'toast ' + (tipo || '');
  el.innerHTML = msg;
  $('#toasts').appendChild(el);
  setTimeout(function () { el.remove(); }, ms || 4200);
}

function modal(op) {
  var fondo = document.createElement('div');
  fondo.className = 'modal-fondo';
  fondo.innerHTML = '<div class="modal ' + (op.ancho || '') + '" role="dialog" aria-modal="true" aria-label="' + esc(op.titulo) + '">' +
    '<div class="cab">' + (op.icono ? ico(op.icono) : '') + '<h3>' + esc(op.titulo) + '</h3><button data-cerrar aria-label="Cerrar">' + ico('x') + '</button></div>' +
    '<div class="cuerpo">' + (op.cuerpo || '') + '</div>' +
    (op.pie !== false ? '<div class="pie">' + (op.pie || '<button class="btn" data-cerrar>Cerrar</button>') + '</div>' : '') + '</div>';
  $('#modales').appendChild(fondo);
  var api = {
    el: fondo.querySelector('.modal'),
    cerrar: function () { fondo.remove(); document.removeEventListener('keydown', esc_); if (op.alCerrar) op.alCerrar(); }
  };
  function esc_(e) { if (e.key === 'Escape') api.cerrar(); }
  document.addEventListener('keydown', esc_);
  fondo.addEventListener('mousedown', function (e) { if (e.target === fondo) api.cerrar(); });
  $$('[data-cerrar]', fondo).forEach(function (b) { b.addEventListener('click', api.cerrar); });
  if (op.alAbrir) op.alAbrir(api.el, api);
  var f = api.el.querySelector('input:not([type=hidden]):not([disabled]), select, textarea');
  if (f && op.enfocar !== false) setTimeout(function () { f.focus(); }, 30);
  return api;
}

function confirmar(mensaje, textoOk, peligro) {
  return new Promise(function (resolve) {
    var r = false;
    var m = modal({
      titulo: 'Confirmar', icono: 'alert', cuerpo: '<p style="margin:0">' + mensaje + '</p>',
      pie: '<button class="btn" data-cerrar>Cancelar</button><button class="btn ' + (peligro ? 'peligro' : 'primario') + '" data-ok>' + esc(textoOk || 'Aceptar') + '</button>',
      alCerrar: function () { resolve(r); }
    });
    m.el.querySelector('[data-ok]').addEventListener('click', function () { r = true; m.cerrar(); });
  });
}

function pedirTexto(titulo, etiqueta, valor) {
  return new Promise(function (resolve) {
    var r = null;
    var m = modal({
      titulo: titulo, cuerpo: '<div class="campo"><label>' + esc(etiqueta) + '</label><input type="text" id="pt" value="' + esc(valor || '') + '"></div>',
      pie: '<button class="btn" data-cerrar>Cancelar</button><button class="btn primario" data-ok>Aceptar</button>',
      alCerrar: function () { resolve(r); }
    });
    function ok() { r = m.el.querySelector('#pt').value.trim(); m.cerrar(); }
    m.el.querySelector('[data-ok]').addEventListener('click', ok);
    m.el.querySelector('#pt').addEventListener('keydown', function (e) { if (e.key === 'Enter') ok(); });
  });
}

function descargarBlob(blob, nombre) {
  if (typeof saveAs === 'function') { saveAs(blob, nombre); return; }
  var a = document.createElement('a'); a.href = URL.createObjectURL(blob); a.download = nombre;
  document.body.appendChild(a); a.click(); setTimeout(function () { URL.revokeObjectURL(a.href); a.remove(); }, 500);
}
function elegirArchivo(accept) {
  return new Promise(function (resolve) {
    var inp = $('#archivoOculto');
    inp.value = ''; inp.accept = accept || '';
    inp.onchange = function () { resolve(inp.files[0] || null); };
    inp.click();
  });
}
function leerArchivo(file, como) {
  return new Promise(function (resolve, reject) {
    var r = new FileReader();
    r.onload = function () { resolve(r.result); };
    r.onerror = reject;
    if (como === 'texto') r.readAsText(file, 'utf-8');
    else if (como === 'dataURL') r.readAsDataURL(file);
    else r.readAsArrayBuffer(file);
  });
}
function nombreArchivo(t) { return String(t || '').normalize('NFD').replace(/[̀-ͯ]/g, '').replace(/[\\/:*?"<>|]/g, '_').replace(/\s+/g, '_').slice(0, 80); }
function libsOk(lista) {
  var falta = lista.filter(function (l) { return !window[l] && !(l === 'jspdf' && window.jspdf); });
  if (falta.length) { toast('No se cargaron las librerías: ' + falta.join(', ') + '. Revisa tu conexión a internet.', 'error'); return false; }
  return true;
}

/* ============================================================
   ESTADO Y PERSISTENCIA
   Modo servidor: GET/PUT api/datos (ver servidor.js).
   Modo archivo:  datos_cotizador.json junto a este HTML (solo lectura).
   Modo local:    datos de ejemplo en memoria; usa "Descargar respaldo".
   ============================================================ */
var D = null;                 // catálogo y documentos
var Persistencia = { modo: 'local', pendiente: false, ultimoGuardado: null, error: null };

function migrar(d) {
  var base = datosIniciales();
  Object.keys(base).forEach(function (k) { if (d[k] == null) d[k] = base[k]; });
  ['parametros', 'folios', 'empresa'].forEach(function (k) { Object.keys(base[k]).forEach(function (s) { if (d[k][s] == null) d[k][s] = base[k][s]; }); });
  if (!d.parametros.reglaGrupo) d.parametros.reglaGrupo = base.parametros.reglaGrupo;
  if (!d.parametros.ruteo) d.parametros.ruteo = base.parametros.ruteo;
  (d.vehiculos || []).forEach(function (v) {
    var b = base.vehiculos[0];
    ['dimensiones', 'neumaticos', 'combustible', 'descanso', 'mantenimiento', 'pagoOperador'].forEach(function (k) { if (!v[k]) v[k] = clonar(b[k]); });
    ['activos', 'gastosFijos', 'equipamiento'].forEach(function (k) { if (!Array.isArray(v[k])) v[k] = []; });
  });
  return d;
}

async function cargarDatos() {
  try {
    var r = await fetch('api/datos', { cache: 'no-store' });
    if (r.ok && (r.headers.get('content-type') || '').indexOf('json') >= 0) {
      var j = await r.json();
      D = migrar(j && j.version ? j : datosIniciales());
      Persistencia.modo = 'servidor';
      if (!j || !j.version) guardar();
      return;
    }
  } catch (e) { /* sin servidor */ }
  try {
    var r2 = await fetch('datos_cotizador.json', { cache: 'no-store' });
    if (r2.ok) { D = migrar(await r2.json()); Persistencia.modo = 'archivo'; return; }
  } catch (e) { /* sin archivo */ }
  D = datosIniciales();
  Persistencia.modo = 'local';
}

var _guardarServidor = debounce(async function () {
  try {
    var r = await fetch('api/datos', { method: 'PUT', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(D) });
    if (!r.ok) throw new Error('HTTP ' + r.status);
    Persistencia.pendiente = false; Persistencia.error = null; Persistencia.ultimoGuardado = new Date();
  } catch (e) {
    Persistencia.error = e.message; Persistencia.pendiente = true;
    toast('No se pudo guardar en el servidor (' + esc(e.message) + '). Descarga un respaldo para no perder cambios.', 'error');
  }
  pintarEstadoGuardado();
}, 700);

function guardar() {
  Persistencia.pendiente = true;
  if (Persistencia.modo === 'servidor') _guardarServidor();
  pintarEstadoGuardado();
}

function pintarEstadoGuardado() {
  var el = $('#estadoGuardado'); if (!el) return;
  var txt, pend = false;
  if (Persistencia.modo === 'servidor') {
    if (Persistencia.error) { txt = 'Error al guardar · Descargar respaldo'; pend = true; }
    else if (Persistencia.pendiente) txt = 'Guardando…';
    else txt = 'Guardado en servidor';
  } else {
    pend = Persistencia.pendiente;
    txt = pend ? 'Cambios sin respaldo · Descargar' : (Persistencia.modo === 'archivo' ? 'Datos de datos_cotizador.json' : 'Datos de ejemplo (sin servidor)');
  }
  el.className = 'estado-guardado' + (pend ? ' pendiente' : '');
  el.title = Persistencia.modo === 'servidor' ? 'Los cambios se guardan automáticamente en el servidor.' :
    'Sin servidor: los cambios viven solo en esta pestaña. Descarga el respaldo (JSON) o ejecuta servidor.js para guardarlos.';
  el.innerHTML = '<span class="punto"></span><span class="txt">' + esc(txt) + '</span>';
}

function exportarRespaldo() {
  var blob = new Blob([JSON.stringify(D, null, 2)], { type: 'application/json' });
  descargarBlob(blob, 'datos_cotizador.json');
  if (Persistencia.modo !== 'servidor') { Persistencia.pendiente = false; pintarEstadoGuardado(); }
  toast('Respaldo descargado. Colócalo junto al HTML como <b>datos_cotizador.json</b> para cargarlo al abrir.', 'ok', 6000);
}
async function importarRespaldo() {
  var f = await elegirArchivo('.json,application/json');
  if (!f) return;
  try {
    var j = JSON.parse(await leerArchivo(f, 'texto'));
    if (!j || !j.vehiculos) throw new Error('El archivo no tiene la estructura esperada.');
    if (!(await confirmar('Se reemplazarán todos los catálogos y cotizaciones actuales con el respaldo <b>' + esc(f.name) + '</b>.', 'Importar', true))) return;
    D = migrar(j); guardar(); navegar(); toast('Respaldo importado.', 'ok');
  } catch (e) { toast('No se pudo importar: ' + esc(e.message), 'error'); }
}

/* ---------- Usuarios activos y permisos (control de interfaz) ---------- */
function usuarioActivo() { return D.usuarios.find(function (u) { return u.id === D.usuarioActivoId; }) || D.usuarios[0]; }
function rolDe(u) { return D.roles.find(function (r) { return r.id === (u && u.rolId); }) || D.roles[0]; }
function puede(modulo) { var r = rolDe(usuarioActivo()); return !r || r.permisos.indexOf('*') >= 0 || r.permisos.indexOf(modulo) >= 0; }

/* ---------- Navegación ---------- */
var MODULOS = [
  { id: 'inicio', nombre: 'Inicio', icono: 'home', grupo: 'General' },
  { id: 'cotizar', nombre: 'Cotizar servicio', icono: 'globe', grupo: 'General' },
  { id: 'cotizaciones', nombre: 'Cotizaciones', icono: 'fileText', grupo: 'General' },
  { id: 'precios', nombre: 'Lista de precios', icono: 'clipboard', grupo: 'General' },
  { id: 'vehiculos', nombre: 'Vehículos', icono: 'truck', grupo: 'Catálogos' },
  { id: 'equipamiento', nombre: 'Equipamiento', icono: 'tool', grupo: 'Catálogos' },
  { id: 'gastos', nombre: 'Gastos fijos generales', icono: 'dollar', grupo: 'Catálogos' },
  { id: 'clientes', nombre: 'Clientes', icono: 'users', grupo: 'Catálogos' },
  { id: 'plantillas', nombre: 'Plantillas', icono: 'file', grupo: 'Sistema' },
  { id: 'configuracion', nombre: 'Configuración', icono: 'settings', grupo: 'Sistema' },
  { id: 'usuarios', nombre: 'Usuarios y roles', icono: 'shield', grupo: 'Sistema' },
  { id: 'documentacion', nombre: 'Documentación', icono: 'book', grupo: 'Sistema' }
];
var VISTAS = {};   // id -> función de render (se registran en los demás bloques)
var vistaActual = null;

function pintarNav() {
  var html = '', grupo = '';
  MODULOS.forEach(function (m) {
    if (!puede(m.id)) return;
    if (m.grupo !== grupo) { grupo = m.grupo; html += '<div class="grupo">' + esc(grupo) + '</div>'; }
    html += '<a href="#/' + m.id + '" data-mod="' + m.id + '" title="' + esc(m.nombre) + '">' + ico(m.icono) + '<span>' + esc(m.nombre) + '</span></a>';
  });
  $('#nav').innerHTML = html;
  var u = usuarioActivo();
  $('#btnUsuario').innerHTML = '<span class="avatar">' + esc((u.nombre || '?').charAt(0).toUpperCase()) + '</span><span class="nowrap">' + esc(u.nombre) + '</span>' + ico('chevD');
  var logo = D.empresa.logo;
  $('#marcaLogo').innerHTML = logo ? '<img src="' + esc(logo) + '" alt="Logo de ' + esc(D.empresa.nombre) + '">' :
    '<span class="avatar" style="width:34px;height:34px;border-radius:8px;background:var(--accent);color:#fff">' + esc((D.empresa.nombre || 'S').charAt(0)) + '</span>';
  $('#marcaNombre').textContent = D.empresa.nombre || 'Mi empresa';
}

function navegar() {
  var id = (location.hash.replace(/^#\/?/, '').split('?')[0]) || 'inicio';
  var mod = MODULOS.find(function (m) { return m.id === id; });
  if (!mod || !puede(id)) { mod = MODULOS.find(function (m) { return puede(m.id); }) || MODULOS[0]; id = mod.id; }
  vistaActual = id;
  $$('#nav a').forEach(function (a) { a.classList.toggle('activo', a.dataset.mod === id); });
  $('#titulo').innerHTML = '<span>' + esc(D.empresa.nombre || 'Cotizador') + '</span><span class="sep">|</span><span>' + esc(mod.nombre) + '</span>';
  document.title = mod.nombre + ' | Cotizador de fletes';
  var esCot = id === 'cotizar';
  $('#vista').classList.toggle('oculto', esCot);
  $('#vistaCotizar').classList.toggle('oculto', !esCot);
  if (esCot) { VISTAS.cotizar(); return; }
  $('#vista').innerHTML = '';
  $('#vista').scrollTop = 0;
  try { VISTAS[id]($('#vista')); }
  catch (e) { console.error(e); $('#vista').innerHTML = '<div class="aviso error">' + ico('alert') + '<div>Error al mostrar el módulo: ' + esc(e.message) + '</div></div>'; }
}

function menuUsuario() {
  var filas = D.usuarios.filter(function (u) { return u.estatus !== 'baja'; }).map(function (u) {
    return '<button class="btn bloque" style="justify-content:flex-start;margin-bottom:6px" data-u="' + u.id + '"><span class="avatar">' + esc(u.nombre.charAt(0)) + '</span>' + esc(u.nombre) + ' <span class="badge azul" style="margin-left:auto">' + esc(rolDe(u).nombre) + '</span></button>';
  }).join('');
  var m = modal({ titulo: 'Usuario activo', icono: 'user', cuerpo: filas + '<p class="faint" style="font-size:12px;margin:10px 0 0">El usuario activo define qué módulos se muestran y quién elaboró cada cotización. No es un control de acceso seguro: la autenticación debe resolverse en el servidor.</p>' });
  $$('[data-u]', m.el).forEach(function (b) {
    b.addEventListener('click', function () { D.usuarioActivoId = b.dataset.u; guardar(); m.cerrar(); pintarNav(); navegar(); });
  });
}

/* ============================================================
   TABLA DE CATÁLOGO GENÉRICA (edición con doble clic)
   ============================================================ */
function tablaCatalogo(op) {
  // op: { cont, filas(), columnas[], alCambiar(fila,campo,valor), acciones(fila) -> [{txt, icono, fn}], vacio, pie(filas), alClic(fila) }
  var estado = { orden: op.ordenInicial || null, dir: 1, buscar: '' };
  op.cont.innerHTML = '<div class="tabla-wrap"><table class="tabla"><thead></thead><tbody></tbody><tfoot></tfoot></table></div>';
  var tabla = op.cont.querySelector('table');

  function valorCelda(f, c) { return c.valor ? c.valor(f) : f[c.id]; }
  function textoCelda(f, c) {
    var v = valorCelda(f, c);
    if (c.render) return c.render(f);
    if (c.tipo === 'moneda') return fmt.mon(v);
    if (c.tipo === 'numero') return fmt.num(v, c.dec == null ? 2 : c.dec);
    if (c.tipo === 'fecha') return fmt.fechaHora(v);
    if (c.tipo === 'select') { var o = (typeof c.opciones === 'function' ? c.opciones() : c.opciones).find(function (x) { return x.v === v; }); return esc(o ? o.t : v || ''); }
    return esc(v == null ? '' : v);
  }
  function pintar() {
    var filas = op.filas().slice();
    if (estado.buscar) {
      var q = estado.buscar.toLowerCase();
      filas = filas.filter(function (f) { return op.columnas.some(function (c) { var v = valorCelda(f, c); return String(v == null ? '' : v).toLowerCase().indexOf(q) >= 0; }); });
    }
    if (estado.orden) {
      var c = op.columnas.find(function (x) { return x.id === estado.orden; });
      filas.sort(function (a, b) { var x = valorCelda(a, c), y = valorCelda(b, c); if (typeof x === 'number' || typeof y === 'number') return (num(x) - num(y)) * estado.dir; return String(x || '').localeCompare(String(y || ''), 'es') * estado.dir; });
    }
    tabla.querySelector('thead').innerHTML = '<tr>' + op.columnas.map(function (c) {
      var flecha = estado.orden === c.id ? (estado.dir > 0 ? ' ▲' : ' ▼') : '';
      return '<th data-col="' + c.id + '" class="' + (c.tipo === 'moneda' || c.tipo === 'numero' ? 'num' : '') + (c.ordenable === false ? ' sin-orden' : '') + '" style="' + (c.ancho ? 'width:' + c.ancho : '') + '">' + esc(c.titulo) + flecha + '</th>';
    }).join('') + (op.acciones ? '<th class="sin-orden" style="width:44px"></th>' : '') + '</tr>';
    tabla.querySelector('tbody').innerHTML = filas.length ? filas.map(function (f) {
      return '<tr data-id="' + esc(f.id) + '"' + (op.alClic ? ' class="click"' : '') + '>' + op.columnas.map(function (c) {
        var ed = c.editable !== false && op.alCambiar && !c.render ? ' editable' : '';
        return '<td data-col="' + c.id + '" class="' + (c.tipo === 'moneda' || c.tipo === 'numero' ? 'num' : '') + ed + '" ' + (ed ? 'title="Doble clic para editar"' : '') + '>' + textoCelda(f, c) + '</td>';
      }).join('') + (op.acciones ? '<td class="menu-fila"><button class="btn icono fantasma" data-menu aria-label="Acciones">' + ico('dots') + '</button></td>' : '') + '</tr>';
    }).join('') : '<tr><td colspan="' + (op.columnas.length + 1) + '"><div class="vacio-estado">' + (op.vacio || 'No se encontraron resultados') + '</div></td></tr>';
    tabla.querySelector('tfoot').innerHTML = op.pie ? op.pie(filas) : '';
  }
  tabla.addEventListener('click', function (e) {
    var th = e.target.closest('th[data-col]');
    if (th) { var c = op.columnas.find(function (x) { return x.id === th.dataset.col; }); if (c && c.ordenable === false) return; if (estado.orden === th.dataset.col) estado.dir *= -1; else { estado.orden = th.dataset.col; estado.dir = 1; } pintar(); return; }
    var bm = e.target.closest('[data-menu]');
    if (bm) {
      e.stopPropagation();
      $$('.menu-pop').forEach(function (m) { m.remove(); });
      var id = bm.closest('tr').dataset.id; var fila = op.filas().find(function (f) { return String(f.id) === id; });
      var pop = document.createElement('div'); pop.className = 'menu-pop';
      var acc = op.acciones(fila);
      pop.innerHTML = acc.map(function (a, i) { return '<button data-i="' + i + '" class="' + (a.peligro ? 'peligro' : '') + '">' + ico(a.icono || 'chevR') + esc(a.txt) + '</button>'; }).join('');
      bm.parentNode.appendChild(pop);
      pop.addEventListener('click', function (ev) { var b = ev.target.closest('[data-i]'); if (!b) return; pop.remove(); acc[+b.dataset.i].fn(fila); });
      setTimeout(function () { document.addEventListener('click', function cerrar() { pop.remove(); document.removeEventListener('click', cerrar); }); }, 0);
      return;
    }
    var tr = e.target.closest('tbody tr[data-id]');
    if (tr && op.alClic && !e.target.closest('input,select,button')) { op.alClic(op.filas().find(function (f) { return String(f.id) === tr.dataset.id; })); }
  });
  tabla.addEventListener('dblclick', function (e) {
    var td = e.target.closest('td.editable'); if (!td || td.querySelector('input,select')) return;
    var id = td.parentNode.dataset.id; var fila = op.filas().find(function (f) { return String(f.id) === id; });
    var c = op.columnas.find(function (x) { return x.id === td.dataset.col; });
    var v = valorCelda(fila, c);
    if (c.tipo === 'select') {
      var opciones = typeof c.opciones === 'function' ? c.opciones() : c.opciones;
      td.innerHTML = '<select>' + opciones.map(function (o) { return '<option value="' + esc(o.v) + '"' + (o.v === v ? ' selected' : '') + '>' + esc(o.t) + '</option>'; }).join('') + '</select>';
    } else {
      td.innerHTML = '<input type="' + (c.tipo === 'moneda' || c.tipo === 'numero' ? 'number' : 'text') + '" step="any" value="' + esc(v == null ? '' : v) + '">';
    }
    var inp = td.firstChild; inp.focus(); if (inp.select) inp.select();
    var listo = false;
    function confirmarCambio(guardarlo) {
      if (listo) return; listo = true;
      if (guardarlo) {
        var nv = inp.value; if (c.tipo === 'moneda' || c.tipo === 'numero') nv = num(nv);
        op.alCambiar(fila, c.id, nv); guardar();
      }
      pintar();
    }
    inp.addEventListener('blur', function () { confirmarCambio(true); });
    inp.addEventListener('keydown', function (ev) { if (ev.key === 'Enter') confirmarCambio(true); if (ev.key === 'Escape') confirmarCambio(false); });
    if (c.tipo === 'select') inp.addEventListener('change', function () { confirmarCambio(true); });
  });
  pintar();
  return { pintar: pintar, buscar: function (q) { estado.buscar = q; pintar(); } };
}

/* ---------- Importar / exportar catálogos (Excel o CSV) ---------- */
async function exportarCatalogoExcel(nombre, columnas, filas) {
  if (!libsOk(['ExcelJS'])) return;
  var libro = new ExcelJS.Workbook();
  var hoja = libro.addWorksheet(nombre.slice(0, 30));
  hoja.columns = columnas.map(function (c) { return { header: c.titulo, key: c.id, width: c.anchoXls || 22 }; });
  filas.forEach(function (f) { var o = {}; columnas.forEach(function (c) { o[c.id] = c.exportar ? c.exportar(f) : f[c.id]; }); hoja.addRow(o); });
  hoja.getRow(1).font = { bold: true, color: { argb: 'FFFFFFFF' } };
  hoja.getRow(1).fill = { type: 'pattern', pattern: 'solid', fgColor: { argb: 'FF0F4C81' } };
  columnas.forEach(function (c, i) { if (c.tipo === 'moneda') hoja.getColumn(i + 1).numFmt = '$#,##0.00'; });
  var buf = await libro.xlsx.writeBuffer();
  descargarBlob(new Blob([buf], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' }), nombreArchivo(nombre) + '.xlsx');
}

async function leerTablaArchivo(file) {
  var nombre = file.name.toLowerCase();
  if (nombre.endsWith('.csv') || nombre.endsWith('.txt')) {
    var filas = parseCSV(await leerArchivo(file, 'texto'));
    var enc = filas.shift() || [];
    return filas.map(function (f) { var o = {}; enc.forEach(function (h, i) { o[h.trim()] = f[i]; }); return o; });
  }
  if (!libsOk(['ExcelJS'])) return [];
  var libro = new ExcelJS.Workbook();
  await libro.xlsx.load(await leerArchivo(file));
  var hoja = libro.worksheets[0]; var res = []; var enc2 = [];
  hoja.eachRow(function (row, i) {
    var vals = row.values.slice(1).map(function (v) { return v && typeof v === 'object' ? (v.result != null ? v.result : v.text || '') : v; });
    if (i === 1) enc2 = vals.map(function (h) { return String(h || '').trim(); });
    else { var o = {}; enc2.forEach(function (h, j) { o[h] = vals[j]; }); res.push(o); }
  });
  return res;
}

async function importarCatalogo(columnas, crear) {
  var f = await elegirArchivo('.xlsx,.csv');
  if (!f) return 0;
  try {
    var filas = await leerTablaArchivo(f);
    var n = 0;
    filas.forEach(function (r) {
      var o = {};
      columnas.forEach(function (c) {
        var v = r[c.titulo]; if (v == null) v = r[c.id];
        if (v == null) return;
        if (c.tipo === 'moneda' || c.tipo === 'numero') v = num(String(v).replace(/[$,\s]/g, ''));
        if (c.importar) v = c.importar(v);
        o[c.id] = v;
      });
      if (Object.keys(o).length) { crear(o); n++; }
    });
    guardar();
    toast(n + ' registros importados de ' + esc(f.name) + '.', 'ok');
    return n;
  } catch (e) { toast('No se pudo importar: ' + esc(e.message), 'error'); return 0; }
}

/* ---------- CSV ---------- */
function parseCSV(texto) {
  var filas = [], fila = [], valor = '', dentro = false;
  texto = String(texto).replace(/^﻿/, '');
  for (var i = 0; i < texto.length; i++) {
    var ch = texto[i], sig = texto[i + 1];
    if (ch === '"' && dentro && sig === '"') { valor += '"'; i++; }
    else if (ch === '"') dentro = !dentro;
    else if (ch === ',' && !dentro) { fila.push(valor.trim()); valor = ''; }
    else if ((ch === '\n' || ch === '\r') && !dentro) {
      if (valor !== '' || fila.length) { fila.push(valor.trim()); filas.push(fila); fila = []; valor = ''; }
      if (ch === '\r' && sig === '\n') i++;
    } else valor += ch;
  }
  if (valor !== '' || fila.length) { fila.push(valor.trim()); filas.push(fila); }
  return filas;
}

/* ============================================================
   INICIO, COTIZACIONES Y CATÁLOGOS (clientes, equipamiento,
   gastos fijos, vehículos)
   ============================================================ */
var ESTADOS_COT = {
  borrador: { t: 'Borradores', uno: 'Borrador', clase: '', color: '#6f757e', icono: 'edit' },
  enviada: { t: 'Enviadas pendientes', uno: 'Enviada', clase: 'ambar', color: '#c98500', icono: 'hourglass' },
  aceptada: { t: 'Aceptadas', uno: 'Aceptada', clase: 'verde', color: '#22a35a', icono: 'checkCircle' },
  rechazada: { t: 'Rechazadas', uno: 'Rechazada', clase: 'rojo', color: '#e05252', icono: 'xCircle' },
  vencida: { t: 'Vencidas', uno: 'Vencida', clase: '', color: '#454a52', icono: 'calendar' }
};
function estadoCot(c) { if ((c.estado === 'borrador' || c.estado === 'enviada') && c.vigencia && c.vigencia < hoyISO()) return 'vencida'; return c.estado; }
function badgeEstado(c) { var e = estadoCot(c); var x = ESTADOS_COT[e] || ESTADOS_COT.borrador; return '<span class="badge ' + x.clase + '">' + ico(x.icono) + esc(x.uno) + '</span>'; }
function totalesCot(c) {
  var sub = (c.items || []).reduce(function (a, i) { return a + num(i.cantidad) * num(i.precioUnitario); }, 0);
  var iva = sub * num(c.ivaPct) / 100;
  return { subtotal: sub, iva: iva, total: sub + iva };
}
function clientePorId(id) { return D.clientes.find(function (c) { return c.id === id; }); }
function nombreCliente(id) { var c = clientePorId(id); return c ? c.nombre : '—'; }
function vehiculoPorId(id) { return D.vehiculos.find(function (v) { return v.id === id; }); }
function siguienteFolio(simular) {
  var f = D.folios, y = new Date().getFullYear();
  var consecutivo = f.consecutivo, anio = f.anio;
  if (f.reinicioAnual && anio !== y) { anio = y; consecutivo = 0; }
  consecutivo++;
  var folio = [f.prefijo, f.incluirAnio ? y : null, String(consecutivo).padStart(4, '0')].filter(function (x) { return x != null && x !== ''; }).join('-');
  if (!simular) { f.consecutivo = consecutivo; f.anio = anio; }
  return folio;
}

/* ---------------- INICIO ---------------- */
VISTAS.inicio = function (cont) {
  var u = usuarioActivo();
  var mesSel = VISTAS.inicio.mes || hoyISO().slice(0, 7);
  cont.innerHTML =
    '<div class="fila entre" style="margin-bottom:16px"><div><h2 style="margin:0;font-size:20px">Hola, ' + esc(u.nombre) + '</h2><div class="muted">Así va tu negocio y esto necesita tu atención.</div></div>' +
    '<div class="fila"><input type="month" id="mesInicio" value="' + mesSel + '" style="width:auto"><button class="btn icono" id="refInicio" title="Actualizar">' + ico('refresh') + '</button>' +
    '<a class="btn primario" href="#/cotizar">' + ico('plus') + 'Nuevo cotizador</a></div></div>' +
    '<div id="cuerpoInicio"></div>';
  $('#mesInicio').addEventListener('change', function (e) { VISTAS.inicio.mes = e.target.value; pintar(); });
  $('#refInicio').addEventListener('click', pintar);
  pintar();

  function pintar() {
    var mes = $('#mesInicio').value || hoyISO().slice(0, 7);
    var y = +mes.slice(0, 4), m = +mes.slice(5, 7);
    var prev = (m === 1 ? (y - 1) + '-12' : y + '-' + String(m - 1).padStart(2, '0'));
    var delMes = D.cotizaciones.filter(function (c) { return (c.fecha || '').slice(0, 7) === mes; });
    var delPrev = D.cotizaciones.filter(function (c) { return (c.fecha || '').slice(0, 7) === prev; });
    var monto = delMes.reduce(function (a, c) { return a + totalesCot(c).subtotal; }, 0);
    function tasa(arr) { return arr.length ? arr.filter(function (c) { return c.estado === 'aceptada'; }).length / arr.length : null; }
    var t = tasa(delMes), tp = tasa(delPrev);
    var conteo = {}; Object.keys(ESTADOS_COT).forEach(function (k) { conteo[k] = 0; });
    delMes.forEach(function (c) { conteo[estadoCot(c)]++; });
    var total = delMes.length || 1;
    var barra = Object.keys(ESTADOS_COT).map(function (k) { return conteo[k] ? '<div style="width:' + (conteo[k] / total * 100) + '%;background:' + ESTADOS_COT[k].color + '" title="' + ESTADOS_COT[k].t + ': ' + conteo[k] + '"></div>' : ''; }).join('');
    var hoy = hoyISO(), en7 = sumarDias(hoy, 7);
    var atender = D.cotizaciones.filter(function (c) {
      var e = estadoCot(c);
      return (e === 'enviada') || ((e === 'borrador' || e === 'enviada') && c.vigencia >= hoy && c.vigencia <= en7);
    }).sort(function (a, b) { return (a.vigencia || '').localeCompare(b.vigencia || ''); });
    var porCliente = {};
    delMes.forEach(function (c) { var k = c.clienteId || '-'; porCliente[k] = porCliente[k] || { n: 0, monto: 0, ac: 0 }; porCliente[k].n++; porCliente[k].monto += totalesCot(c).subtotal; if (c.estado === 'aceptada') porCliente[k].ac++; });
    var top = Object.keys(porCliente).map(function (k) { return { id: k, d: porCliente[k] }; }).sort(function (a, b) { return b.d.monto - a.d.monto; }).slice(0, 5);

    $('#cuerpoInicio').innerHTML =
      '<div class="tarjeta" style="margin-bottom:14px"><div class="cab"><span class="icono-cab">' + ico('chart') + '</span><h2>Resumen del mes</h2></div><div class="cuerpo">' +
      '<div class="kpis">' +
      '<div class="kpi"><div class="t">' + ico('fileText') + 'Cotizaciones</div><div class="v">' + delMes.length + '</div><div class="s">emitidas en el mes</div></div>' +
      '<div class="kpi"><div class="t">' + ico('dollar') + 'Monto cotizado</div><div class="v">' + fmt.mon(monto, 0) + '</div><div class="s">sin impuestos</div></div>' +
      '<div class="kpi"><div class="t">' + ico('target') + 'Tasa de cierre</div><div class="v">' + (t == null ? 'Sin datos' : fmt.pct(t * 100, 0)) + '</div><div class="s">' + (tp == null ? 'sin mes anterior para comparar' : 'mes anterior: ' + fmt.pct(tp * 100, 0)) + '</div></div>' +
      '</div>' +
      '<div class="lbl" style="margin-top:16px">Estado de las cotizaciones</div><div class="barra-estados" role="img" aria-label="Distribución por estado">' + barra + '</div>' +
      '<div class="leyenda-estados">' + Object.keys(ESTADOS_COT).map(function (k) { return '<div><span class="dot" style="background:' + ESTADOS_COT[k].color + '"></span>' + ico(ESTADOS_COT[k].icono) + esc(ESTADOS_COT[k].t) + '<b>' + conteo[k] + '</b></div>'; }).join('') + '</div>' +
      '<div class="faint" style="font-size:12px;margin-top:12px">' + ico('info') + ' Tasa de cierre = aceptadas entre todas las emitidas en el mes. Montos cotizados, no cobrados.</div>' +
      '</div></div>' +
      '<div class="grid-2">' +
      '<div class="tarjeta"><div class="cab"><span class="icono-cab">' + ico('clock') + '</span><h2>Por atender</h2></div><div class="cuerpo">' +
      '<div class="faint" style="font-size:12px;margin-bottom:8px">' + ico('info') + ' Cotizaciones que vencen esta semana y enviadas sin respuesta registrada.</div>' +
      (atender.length ? '<div class="tabla-wrap"><table class="tabla"><tbody>' + atender.slice(0, 8).map(function (c) {
        return '<tr class="click" data-cot="' + c.id + '"><td class="mono">' + esc(c.folio) + '</td><td>' + esc(nombreCliente(c.clienteId)) + '</td><td>' + badgeEstado(c) + '</td><td class="num">vence ' + fmt.fecha(c.vigencia) + '</td></tr>';
      }).join('') + '</tbody></table></div>' : '<div class="vacio-estado"><div class="circulo">' + ico('check') + '</div><b>Nada por atender</b>No hay cotizaciones por vencer ni enviadas pendientes.</div>') +
      '</div></div>' +
      '<div class="tarjeta"><div class="cab"><span class="icono-cab">' + ico('users') + '</span><h2>Mejores clientes</h2></div><div class="cuerpo">' +
      (top.length ? '<div class="tabla-wrap"><table class="tabla"><thead><tr><th class="sin-orden">Cliente</th><th class="num sin-orden">Cotizaciones</th><th class="num sin-orden">Aceptadas</th><th class="num sin-orden">Monto</th></tr></thead><tbody>' + top.map(function (x) {
        return '<tr><td>' + esc(nombreCliente(x.id)) + '</td><td class="num">' + x.d.n + '</td><td class="num">' + x.d.ac + '</td><td class="num">' + fmt.mon(x.d.monto, 0) + '</td></tr>';
      }).join('') + '</tbody></table></div>' : '<div class="vacio-estado"><div class="circulo">' + ico('users') + '</div><b>Todavía no hay datos</b>Aparecen en cuanto haya cotizaciones en el mes.</div>') +
      '</div></div></div>';
    $$('[data-cot]', cont).forEach(function (tr) { tr.addEventListener('click', function () { abrirCotizacion(D.cotizaciones.find(function (c) { return c.id === tr.dataset.cot; })); }); });
  }
};

/* ---------------- COTIZACIONES ---------------- */
VISTAS.cotizaciones = function (cont) {
  var filtroEstado = '';
  cont.innerHTML = '<div class="barra-acciones"><a class="btn primario" href="#/cotizar">' + ico('route') + 'Nuevo cotizador</a>' +
    '<div class="con-prefijo" style="width:260px"><span>' + ico('search') + '</span><input type="search" id="buscarCot" placeholder="Buscar folio, cliente…"></div>' +
    '<select id="filtroEstado" style="width:auto"><option value="">Todos los estados</option>' + Object.keys(ESTADOS_COT).map(function (k) { return '<option value="' + k + '">' + ESTADOS_COT[k].uno + '</option>'; }).join('') + '</select>' +
    '<div class="espacio"></div><button class="btn" id="expCot">' + ico('download') + 'Exportar</button></div><div id="tablaCot"></div>';
  var t = tablaCatalogo({
    cont: $('#tablaCot'),
    filas: function () { return D.cotizaciones.filter(function (c) { return !filtroEstado || estadoCot(c) === filtroEstado; }); },
    ordenInicial: 'folio',
    columnas: [
      { id: 'folio', titulo: 'Folio', render: function (c) { return '<span class="mono">' + esc(c.folio) + '</span>'; } },
      { id: 'fecha', titulo: 'Fecha', render: function (c) { return fmt.fecha(c.fecha); } },
      { id: 'vigencia', titulo: 'Vigencia', render: function (c) { return fmt.fecha(c.vigencia); } },
      { id: 'cliente', titulo: 'Cliente', valor: function (c) { return nombreCliente(c.clienteId); }, render: function (c) { return esc(nombreCliente(c.clienteId)); } },
      { id: 'estado', titulo: 'Estado', valor: function (c) { return estadoCot(c); }, render: badgeEstado },
      { id: 'total', titulo: 'Total', tipo: 'moneda', valor: function (c) { return totalesCot(c).total; }, render: function (c) { return fmt.mon(totalesCot(c).total); } }
    ],
    alClic: abrirCotizacion,
    acciones: function (c) {
      return [
        { txt: 'Abrir', icono: 'eye', fn: abrirCotizacion },
        { txt: 'Exportar a Word', icono: 'word', fn: function (x) { exportarCotizacionWord(x); } },
        { txt: 'Exportar a PDF', icono: 'pdf', fn: function (x) { exportarCotizacionPDF(x); } },
        { txt: 'Duplicar', icono: 'copy', fn: duplicarCotizacion },
        { txt: 'Eliminar', icono: 'trash', peligro: true, fn: eliminarCotizacion }
      ];
    },
    vacio: 'No se encontraron resultados'
  });
  $('#buscarCot').addEventListener('input', function (e) { t.buscar(e.target.value); });
  $('#filtroEstado').addEventListener('change', function (e) { filtroEstado = e.target.value; t.pintar(); });
  $('#expCot').addEventListener('click', function () {
    exportarCatalogoExcel('Cotizaciones', [
      { id: 'folio', titulo: 'Folio' }, { id: 'fecha', titulo: 'Fecha' }, { id: 'vigencia', titulo: 'Vigencia' },
      { id: 'cliente', titulo: 'Cliente', exportar: function (c) { return nombreCliente(c.clienteId); } },
      { id: 'estado', titulo: 'Estado', exportar: function (c) { return ESTADOS_COT[estadoCot(c)].uno; } },
      { id: 'subtotal', titulo: 'Subtotal', tipo: 'moneda', exportar: function (c) { return totalesCot(c).subtotal; } },
      { id: 'total', titulo: 'Total', tipo: 'moneda', exportar: function (c) { return totalesCot(c).total; } }
    ], D.cotizaciones);
  });
};

function duplicarCotizacion(c) {
  var n = clonar(c); n.id = uid('cot'); n.folio = siguienteFolio(); n.fecha = hoyISO(); n.vigencia = sumarDias(hoyISO(), num(D.parametros.vigenciaDias) || 30); n.estado = 'borrador'; n.creado = new Date().toISOString();
  D.cotizaciones.push(n); guardar(); toast('Cotización duplicada como <b>' + esc(n.folio) + '</b>.', 'ok');
  if (vistaActual === 'cotizaciones') navegar();
}
async function eliminarCotizacion(c) {
  if (!(await confirmar('¿Eliminar la cotización <b>' + esc(c.folio) + '</b>? El folio no se reutiliza.', 'Eliminar', true))) return;
  D.cotizaciones = D.cotizaciones.filter(function (x) { return x.id !== c.id; }); guardar(); navegar();
}

function abrirCotizacion(c) {
  if (!c) return;
  var m = modal({
    titulo: 'Cotización ' + c.folio, icono: 'fileText', ancho: 'ancho',
    cuerpo: '<div id="cotDet"></div>',
    pie: '<button class="btn peligro" data-acc="eliminar">' + ico('trash') + 'Eliminar</button><button class="btn" data-acc="duplicar">' + ico('copy') + 'Duplicar</button><div style="flex:1"></div>' +
      '<button class="btn" data-acc="excel">' + ico('table') + 'Excel</button><button class="btn" data-acc="pdf">' + ico('pdf') + 'PDF</button><button class="btn primario" data-acc="word">' + ico('word') + 'Word</button>',
    alCerrar: function () { if (vistaActual === 'cotizaciones' || vistaActual === 'inicio') navegar(); }
  });
  function pintar() {
    var tot = totalesCot(c);
    var u = D.usuarios.find(function (x) { return x.id === c.usuarioId; });
    $('#cotDet', m.el).innerHTML =
      '<div class="grid-4">' +
      '<div class="campo"><label>Folio</label><input type="text" value="' + esc(c.folio) + '" disabled></div>' +
      '<div class="campo"><label>Fecha</label><input type="date" data-f="fecha" value="' + esc(c.fecha) + '"></div>' +
      '<div class="campo"><label>Vigencia hasta</label><input type="date" data-f="vigencia" value="' + esc(c.vigencia) + '"></div>' +
      '<div class="campo"><label>Estado</label><select data-f="estado">' + ['borrador', 'enviada', 'aceptada', 'rechazada'].map(function (k) { return '<option value="' + k + '"' + (c.estado === k ? ' selected' : '') + '>' + ESTADOS_COT[k].uno + '</option>'; }).join('') + '</select></div>' +
      '<div class="campo" style="grid-column:span 2"><label>Cliente</label><select data-f="clienteId"><option value="">— Selecciona —</option>' + D.clientes.map(function (x) { return '<option value="' + x.id + '"' + (x.id === c.clienteId ? ' selected' : '') + '>' + esc(x.nombre) + '</option>'; }).join('') + '</select></div>' +
      '<div class="campo"><label>IVA %</label><input type="number" data-f="ivaPct" value="' + esc(c.ivaPct) + '"></div>' +
      '<div class="campo"><label>Elaboró</label><input type="text" value="' + esc(u ? u.nombre : '—') + '" disabled></div>' +
      '</div>' +
      (estadoCot(c) === 'vencida' ? '<div class="aviso" style="margin-top:12px">' + ico('calendar') + '<div>La vigencia ya pasó. Si sigue en negociación, amplía la fecha de vigencia.</div></div>' : '') +
      '<div class="tabla-wrap" style="margin-top:14px"><table class="tabla"><thead><tr><th class="sin-orden" style="width:40px">No.</th><th class="sin-orden">Descripción</th><th class="sin-orden num" style="width:90px">Cant.</th><th class="sin-orden num" style="width:150px">P. unitario</th><th class="sin-orden num" style="width:140px">Importe</th><th class="sin-orden" style="width:80px"></th></tr></thead><tbody>' +
      (c.items || []).map(function (it, i) {
        return '<tr data-i="' + i + '"><td>' + (i + 1) + '</td><td><input type="text" data-it="descripcion" value="' + esc(it.descripcion) + '"></td><td><input type="number" min="0" step="any" data-it="cantidad" value="' + esc(it.cantidad) + '"></td><td><input type="number" step="any" data-it="precioUnitario" value="' + esc(Motor.redondear(it.precioUnitario, 2)) + '"></td><td class="num">' + fmt.mon(num(it.cantidad) * num(it.precioUnitario)) + '</td><td class="nowrap">' +
          (it.snapshot ? '<button class="btn icono fantasma" data-ver="' + i + '" title="Ver cálculo guardado">' + ico('eye') + '</button>' : '') +
          '<button class="btn icono fantasma peligro" data-quitar="' + i + '" title="Quitar renglón">' + ico('trash') + '</button></td></tr>';
      }).join('') +
      '</tbody><tfoot><tr><td colspan="4" class="num">Subtotal</td><td class="num">' + fmt.mon(tot.subtotal) + '</td><td></td></tr>' +
      '<tr><td colspan="4" class="num">IVA (' + fmt.num(c.ivaPct, 0) + '%)</td><td class="num">' + fmt.mon(tot.iva) + '</td><td></td></tr>' +
      '<tr><td colspan="4" class="num">Total</td><td class="num">' + fmt.mon(tot.total) + '</td><td></td></tr></tfoot></table></div>' +
      '<div class="fila" style="margin-top:8px"><button class="btn chico" id="agregarRenglon">' + ico('plus') + 'Agregar renglón manual</button><a class="btn chico suave" href="#/cotizar" data-cerrar-ir>' + ico('route') + 'Agregar un viaje desde el cotizador</a></div>' +
      '<div class="campo" style="margin-top:14px"><label>Notas</label><textarea data-f="notas" placeholder="Opcional. Aparecerá en la cotización al exportar.">' + esc(c.notas || '') + '</textarea></div>';
    $$('[data-f]', m.el).forEach(function (el) {
      el.addEventListener('change', function () { c[el.dataset.f] = el.type === 'number' ? num(el.value) : el.value; guardar(); pintar(); });
    });
    $$('[data-it]', m.el).forEach(function (el) {
      el.addEventListener('change', function () { var i = +el.closest('tr').dataset.i; c.items[i][el.dataset.it] = el.type === 'number' ? num(el.value) : el.value; guardar(); pintar(); });
    });
    $$('[data-quitar]', m.el).forEach(function (b) { b.addEventListener('click', function () { c.items.splice(+b.dataset.quitar, 1); guardar(); pintar(); }); });
    $$('[data-ver]', m.el).forEach(function (b) { b.addEventListener('click', function () { verCalculoGuardado(c.items[+b.dataset.ver], m); }); });
    $('#agregarRenglon', m.el).addEventListener('click', function () { c.items.push({ id: uid('it'), descripcion: 'Concepto', cantidad: 1, precioUnitario: 0 }); guardar(); pintar(); });
    var ir = m.el.querySelector('[data-cerrar-ir]'); if (ir) ir.addEventListener('click', function () { Cot.cotizacionDestinoId = c.id; m.cerrar(); });
  }
  pintar();
  $$('[data-acc]', m.el).forEach(function (b) {
    b.addEventListener('click', function () {
      var a = b.dataset.acc;
      if (a === 'eliminar') { m.cerrar(); eliminarCotizacion(c); }
      if (a === 'duplicar') { m.cerrar(); duplicarCotizacion(c); }
      if (a === 'excel') exportarCotizacionExcel(c);
      if (a === 'pdf') exportarCotizacionPDF(c);
      if (a === 'word') exportarCotizacionWord(c);
    });
  });
}

function verCalculoGuardado(item, modalPadre) {
  var s = item.snapshot; if (!s) return;
  var r = s.resultado;
  var filas = s.tipo === 'renta' ? [
    ['Vehículo', esc(s.vehiculoNombre)], ['Periodo', esc(s.periodoTexto || '')], ['Disponibilidad (fijo)', fmt.mon(r.disponibilidad)], ['Uso (variable)', fmt.mon(r.uso)],
    ['Costo base', fmt.mon(r.costoBase)], ['Utilidad', fmt.mon(r.utilidad)], ['Subtotal', fmt.mon(r.subtotal)]
  ] : [
    ['Vehículo', esc(s.vehiculoNombre)], ['Ruta', esc(s.rutaTexto || '')], ['Distancia', fmt.km(r.kmTotal) + ' (cargado ' + fmt.km(r.kmCargado) + ')'], ['Tiempo de servicio', fmt.dur(r.servicioH)],
    ['Gasto total', fmt.mon(r.gastoTotal)], ['Peajes', fmt.mon(r.peajes)], ['Utilidad', fmt.mon(r.utilidad)], ['Financiamiento', fmt.mon(r.financiamiento)], ['Aumento por carga', fmt.mon(r.aumentoCarga)], ['Subtotal cliente', fmt.mon(r.subtotalCliente)], ['Total con IVA', fmt.mon(r.totalConIva)]
  ];
  var m = modal({
    titulo: 'Cálculo guardado · ' + fmt.fechaHora(s.fecha), icono: 'database', ancho: 'medio',
    cuerpo: '<div class="aviso info" style="margin-bottom:12px">' + ico('info') + '<div>Copia fija del cálculo al momento de cotizar. Para cotizar con los datos de hoy, ábrelo en el cotizador: se recalcula con los parámetros actuales del vehículo.</div></div>' +
      '<div class="tabla-wrap"><table class="tabla"><tbody>' + filas.map(function (f) { return '<tr><td class="muted">' + f[0] + '</td><td class="num">' + f[1] + '</td></tr>'; }).join('') + '</tbody></table></div>',
    pie: '<button class="btn" data-cerrar>Cerrar</button>' + (s.entrada ? '<button class="btn primario" data-abrir>' + ico('refresh') + 'Abrir en el cotizador (datos de hoy)</button>' : '')
  });
  var b = m.el.querySelector('[data-abrir]');
  if (b) b.addEventListener('click', function () { m.cerrar(); if (modalPadre) modalPadre.cerrar(); cargarSnapshotEnCotizador(s); location.hash = '#/cotizar'; });
}

/* ---------------- CLIENTES ---------------- */
var COLS_CLIENTE = [
  { id: 'nombre', titulo: 'Nombre comercial' },
  { id: 'legalName', titulo: 'Razón social' },
  { id: 'rfc', titulo: 'RFC' },
  { id: 'contacto', titulo: 'Contacto' },
  { id: 'email', titulo: 'Email' },
  { id: 'telefono', titulo: 'Teléfono' },
  { id: 'direccion', titulo: 'Dirección' },
  { id: 'tipo', titulo: 'Tipo', tipo: 'select', opciones: [{ v: 'externo', t: 'Externo' }, { v: 'grupo', t: 'Grupo Servifácil' }], importar: function (v) { return /grupo/i.test(v) ? 'grupo' : 'externo'; }, exportar: function (c) { return c.tipo === 'grupo' ? 'Grupo Servifácil' : 'Externo'; } }
];
function nuevoCliente(datos) { var c = Object.assign({ id: uid('cli'), nombre: '', legalName: '', rfc: '', contacto: '', email: '', telefono: '', direccion: '', tipo: 'externo', creado: new Date().toISOString() }, datos || {}); D.clientes.push(c); return c; }
function formularioCliente(alGuardar) {
  var m = modal({
    titulo: 'Nuevo cliente', icono: 'users', ancho: 'medio',
    cuerpo: '<div class="grid-2">' + COLS_CLIENTE.map(function (c) {
      if (c.tipo === 'select') return '<div class="campo"><label>' + c.titulo + '</label><select data-c="' + c.id + '">' + c.opciones.map(function (o) { return '<option value="' + o.v + '">' + o.t + '</option>'; }).join('') + '</select></div>';
      return '<div class="campo"><label>' + c.titulo + (c.id === 'nombre' ? ' *' : '') + '</label><input type="text" data-c="' + c.id + '"></div>';
    }).join('') + '</div><p class="faint" style="font-size:12px">El tipo «Grupo Servifácil» activa la regla comercial interna (Configuración → Regla comercial del grupo).</p>',
    pie: '<button class="btn" data-cerrar>Cancelar</button><button class="btn primario" data-ok>' + ico('save') + 'Guardar cliente</button>'
  });
  m.el.querySelector('[data-ok]').addEventListener('click', function () {
    var o = {}; $$('[data-c]', m.el).forEach(function (el) { o[el.dataset.c] = el.value.trim(); });
    if (!o.nombre) { m.el.querySelector('[data-c=nombre]').classList.add('invalido'); toast('El nombre es obligatorio.', 'warn'); return; }
    var c = nuevoCliente(o); guardar(); m.cerrar(); if (alGuardar) alGuardar(c);
  });
}
VISTAS.clientes = function (cont) {
  cont.innerHTML = '<div class="barra-acciones"><button class="btn primario" id="nuevoCli">' + ico('plus') + 'Nuevo cliente</button><button class="btn" id="impCli">' + ico('upload') + 'Importar</button><button class="btn" id="expCli">' + ico('download') + 'Exportar</button>' +
    '<div class="con-prefijo" style="width:260px"><span>' + ico('search') + '</span><input type="search" id="busCli" placeholder="Agregar filtro: buscar…"></div></div><div id="tablaCli"></div>';
  var t = tablaCatalogo({
    cont: $('#tablaCli'), filas: function () { return D.clientes; }, columnas: COLS_CLIENTE, ordenInicial: 'nombre',
    alCambiar: function (f, campo, v) { f[campo] = v; },
    acciones: function () { return [{ txt: 'Eliminar', icono: 'trash', peligro: true, fn: async function (f) { if (await confirmar('¿Eliminar el cliente <b>' + esc(f.nombre) + '</b>? Las cotizaciones existentes conservarán el renglón, pero sin datos del cliente.', 'Eliminar', true)) { D.clientes = D.clientes.filter(function (x) { return x !== f; }); guardar(); t.pintar(); } } }]; },
    vacio: '<div class="circulo">' + ico('users') + '</div><b>Sin clientes</b>Registra tu directorio para cotizar y exportar con sus datos.'
  });
  $('#nuevoCli').addEventListener('click', function () { formularioCliente(function () { t.pintar(); }); });
  $('#busCli').addEventListener('input', function (e) { t.buscar(e.target.value); });
  $('#expCli').addEventListener('click', function () { exportarCatalogoExcel('Clientes', COLS_CLIENTE, D.clientes); });
  $('#impCli').addEventListener('click', async function () { await importarCatalogo(COLS_CLIENTE, function (o) { if (o.nombre) nuevoCliente(o); }); t.pintar(); });
};

/* ---------------- EQUIPAMIENTO ---------------- */
var COLS_EQUIPO = [
  { id: 'equipo', titulo: 'Equipo', anchoXls: 40 },
  { id: 'descripcion', titulo: 'Descripción', anchoXls: 36 },
  { id: 'precio', titulo: 'Precio', tipo: 'moneda', anchoXls: 14 },
  { id: 'creado', titulo: 'Fecha de creación', tipo: 'fecha', editable: false, exportar: function (f) { return fmt.fechaHora(f.creado); } }
];
VISTAS.equipamiento = function (cont) {
  cont.innerHTML = '<div class="aviso info" style="margin-bottom:12px">' + ico('info') + '<div>Captura el <b>precio sin IVA</b>. Cada vehículo indica cuántas piezas lleva y cada cuánto las reemplaza; con eso se calcula el costo por día y por hora que se cobra en cada viaje.</div></div>' +
    '<div class="barra-acciones"><button class="btn primario" id="nuevoEq">' + ico('plus') + 'Nuevo registro</button><button class="btn" id="impEq">' + ico('upload') + 'Importar</button><button class="btn" id="expEq">' + ico('download') + 'Exportar</button>' +
    '<div class="con-prefijo" style="width:260px"><span>' + ico('search') + '</span><input type="search" id="busEq" placeholder="Agregar filtro: buscar…"></div></div><div id="tablaEq"></div>';
  var t = tablaCatalogo({
    cont: $('#tablaEq'), filas: function () { return D.equipamiento; }, columnas: COLS_EQUIPO,
    alCambiar: function (f, campo, v) { f[campo] = v; },
    acciones: function () { return [{ txt: 'Eliminar', icono: 'trash', peligro: true, fn: async function (f) { var usos = D.vehiculos.filter(function (v) { return v.equipamiento.some(function (e) { return e.equipoId === f.id; }); }).length; if (await confirmar('¿Eliminar <b>' + esc(f.equipo) + '</b>?' + (usos ? ' Está asignado a ' + usos + ' vehículo(s) y dejará de costearse.' : ''), 'Eliminar', true)) { D.equipamiento = D.equipamiento.filter(function (x) { return x !== f; }); guardar(); t.pintar(); } } }]; }
  });
  $('#nuevoEq').addEventListener('click', function () { D.equipamiento.unshift({ id: uid('eq'), equipo: 'Nuevo equipo', descripcion: '', precio: 0, creado: new Date().toISOString() }); guardar(); t.pintar(); toast('Registro agregado. Doble clic en cada celda para editarla.'); });
  $('#busEq').addEventListener('input', function (e) { t.buscar(e.target.value); });
  $('#expEq').addEventListener('click', function () { exportarCatalogoExcel('Equipamiento', COLS_EQUIPO, D.equipamiento); });
  $('#impEq').addEventListener('click', async function () { await importarCatalogo(COLS_EQUIPO.slice(0, 3), function (o) { if (o.equipo) D.equipamiento.push(Object.assign({ id: uid('eq'), creado: new Date().toISOString(), descripcion: '', precio: 0 }, o)); }); t.pintar(); });
};

/* ---------------- GASTOS FIJOS GENERALES ---------------- */
var OPC_PERIODO = [{ v: 'diario', t: 'Diario' }, { v: 'semanal', t: 'Semanal' }, { v: 'quincenal', t: 'Quincenal' }, { v: 'mensual', t: 'Mensual' }, { v: 'bimestral', t: 'Bimestral' }, { v: 'trimestral', t: 'Trimestral' }, { v: 'semestral', t: 'Semestral' }, { v: 'anual', t: 'Anual' }];
var COLS_GASTO = [
  { id: 'nombre', titulo: 'Nombre', anchoXls: 40 },
  { id: 'periodo', titulo: 'Periodo de pago', tipo: 'select', opciones: OPC_PERIODO, importar: function (v) { var x = OPC_PERIODO.find(function (o) { return o.t.toLowerCase() === String(v).toLowerCase() || o.v === String(v).toLowerCase(); }); return x ? x.v : 'mensual'; }, exportar: function (f) { var o = OPC_PERIODO.find(function (x) { return x.v === f.periodo; }); return o ? o.t : f.periodo; } },
  { id: 'monto', titulo: 'Monto', tipo: 'moneda' },
  { id: 'anual', titulo: 'Anualizado', tipo: 'moneda', editable: false, valor: function (f) { return Motor.anualizar(f.monto, f.periodo); }, exportar: function (f) { return Motor.anualizar(f.monto, f.periodo); } },
  { id: 'descripcion', titulo: 'Descripción', anchoXls: 40 },
  { id: 'creado', titulo: 'Creado', tipo: 'fecha', editable: false, exportar: function (f) { return fmt.fechaHora(f.creado); } }
];
VISTAS.gastos = function (cont) {
  var tab = VISTAS.gastos.tab || 'listado';
  cont.innerHTML = '<div class="pestanas"><button data-tab="listado">' + ico('clipboard') + 'Listado</button><button data-tab="analisis">' + ico('chart') + 'Análisis de datos</button></div><div id="gastosCuerpo"></div>';
  $$('[data-tab]', cont).forEach(function (b) { b.classList.toggle('activa', b.dataset.tab === tab); b.addEventListener('click', function () { VISTAS.gastos.tab = b.dataset.tab; VISTAS.gastos(cont); }); });
  var cuerpo = $('#gastosCuerpo');
  if (tab === 'listado') {
    cuerpo.innerHTML = '<div class="barra-acciones"><button class="btn primario" id="nuevoGf">' + ico('plus') + 'Nuevo registro</button><button class="btn" id="impGf">' + ico('upload') + 'Importar</button><button class="btn" id="expGf">' + ico('download') + 'Exportar</button>' +
      '<div class="con-prefijo" style="width:260px"><span>' + ico('search') + '</span><input type="search" id="busGf" placeholder="Agregar filtro: buscar…"></div></div><div id="tablaGf"></div>';
    var t = tablaCatalogo({
      cont: $('#tablaGf'), filas: function () { return D.gastosFijos; }, columnas: COLS_GASTO,
      alCambiar: function (f, campo, v) { f[campo] = v; },
      acciones: function () { return [{ txt: 'Eliminar', icono: 'trash', peligro: true, fn: async function (f) { if (await confirmar('¿Eliminar <b>' + esc(f.nombre) + '</b>?', 'Eliminar', true)) { D.gastosFijos = D.gastosFijos.filter(function (x) { return x !== f; }); guardar(); t.pintar(); } } }]; },
      pie: function (filas) { var s = filas.reduce(function (a, f) { return a + Motor.anualizar(f.monto, f.periodo); }, 0); return '<tr><td colspan="3">Total anualizado (' + filas.length + ' conceptos)</td><td class="num">' + fmt.mon(s) + '</td><td colspan="3"></td></tr>'; }
    });
    $('#nuevoGf').addEventListener('click', function () { D.gastosFijos.unshift({ id: uid('gf'), nombre: 'Nuevo gasto', periodo: 'mensual', monto: 0, descripcion: '', creado: new Date().toISOString() }); guardar(); t.pintar(); toast('Registro agregado. Doble clic en cada celda para editarla.'); });
    $('#busGf').addEventListener('input', function (e) { t.buscar(e.target.value); });
    $('#expGf').addEventListener('click', function () { exportarCatalogoExcel('Gastos fijos generales', COLS_GASTO, D.gastosFijos); });
    $('#impGf').addEventListener('click', async function () { await importarCatalogo([COLS_GASTO[0], COLS_GASTO[1], COLS_GASTO[2], COLS_GASTO[4]], function (o) { if (o.nombre) D.gastosFijos.push(Object.assign({ id: uid('gf'), periodo: 'mensual', monto: 0, descripcion: '', creado: new Date().toISOString() }, o)); }); t.pintar(); });
    return;
  }
  // Análisis de datos
  function pintarAnalisis() {
    var tr = Motor.tarifaRecuperacion(D);
    cuerpo.innerHTML =
      '<div class="tarjeta" style="margin-bottom:14px"><div class="cab"><h2>Tarifa de recuperación</h2></div><div class="cuerpo"><div class="grid-4">' +
      '<div class="kpi"><div class="t">GASTOS FIJOS ANUALES</div><div class="v">' + fmt.mon(tr.gastoAnual) + '</div><div class="s">Suma anualizada de tu catálogo</div></div>' +
      '<div class="kpi"><div class="t">VEHÍCULOS</div><div class="v">' + fmt.ent(tr.vehiculos) + '</div><div class="s">Total de la flota</div></div>' +
      '<div class="kpi"><div class="t">HORAS FACTURABLES / AÑO</div><div class="v">' + fmt.ent(tr.horasFlota) + '</div><div class="s">Σ (días × horas × vehículos)</div></div>' +
      '<div class="kpi" style="border-color:var(--accent);background:var(--accent-soft)"><div class="t" style="color:var(--accent)">TARIFA DE RECUPERACIÓN / HORA</div><div class="v" style="color:var(--accent)">' + fmt.mon(tr.tarifaHora) + ' <small style="font-size:13px">/ hr</small></div><div class="s">' + fmt.mon(tr.gastoAnual) + ' ÷ ' + fmt.ent(tr.horasFlota) + ' hr</div></div>' +
      '</div></div></div>' +
      '<div class="tarjeta"><div class="cab"><h2>Tiempo efectivo</h2><div style="flex:1"></div><button class="btn chico suave" id="nuevaCat">' + ico('plus') + 'Categoría</button></div><div class="cuerpo">' +
      '<div class="aviso" style="margin-bottom:12px">' + ico('alert') + '<div>Usa <b>horas realmente facturables</b>, no las horas que el camión está disponible o encendido. Ej.: si la unidad está disponible 12 h al día pero en promedio solo factura 8, captura 8. Sobrestimar estas horas baja artificialmente tu tarifa y hace que subrecuperes tus gastos fijos durante todo el año.</div></div>' +
      '<div class="tabla-wrap"><table class="tabla"><thead><tr><th class="sin-orden">Categoría de vehículo</th><th class="sin-orden">Vehículos</th><th class="sin-orden">Días / año</th><th class="sin-orden">Horas / día</th><th class="sin-orden num" style="color:var(--accent)">Total horas / año</th><th class="sin-orden"></th></tr></thead><tbody>' +
      D.tiempoEfectivo.map(function (c, i) {
        function st(campo, max) { return '<div class="stepper" style="width:170px"><button data-paso="-1" data-i="' + i + '" data-campo="' + campo + '" aria-label="Disminuir">−</button><input type="number" min="0" max="' + max + '" data-i="' + i + '" data-campo="' + campo + '" value="' + c[campo] + '"><button data-paso="1" data-i="' + i + '" data-campo="' + campo + '" aria-label="Aumentar">+</button></div>'; }
        var usada = D.vehiculos.some(function (v) { return v.categoriaId === c.id; });
        return '<tr><td><input type="text" data-i="' + i + '" data-campo="nombre" value="' + esc(c.nombre) + '" style="max-width:180px"></td><td>' + st('vehiculos', 9999) + '</td><td>' + st('dias', 366) + '</td><td>' + st('horasDia', 24) + '</td><td class="num" style="color:var(--accent);font-weight:600">' + fmt.ent(c.vehiculos * c.dias * c.horasDia) + '</td><td>' + (usada ? '' : '<button class="btn icono fantasma peligro" data-borrar="' + i + '" title="Eliminar categoría">' + ico('trash') + '</button>') + '</td></tr>';
      }).join('') +
      '</tbody><tfoot><tr><td>Total flota</td><td>' + fmt.ent(tr.vehiculos) + '</td><td>—</td><td>—</td><td class="num" style="color:var(--accent)">' + fmt.ent(tr.horasFlota) + '</td><td></td></tr></tfoot></table></div>' +
      '<p class="faint" style="font-size:12px;margin:10px 0 0">Cada perfil de vehículo elige su categoría: de ahí salen las horas al año con las que se reparten sus gastos fijos específicos, su equipamiento y su costo de capital. Los gastos fijos generales se reparten entre las horas de toda la flota.</p>' +
      '</div></div>';
    $$('[data-paso]', cuerpo).forEach(function (b) { b.addEventListener('click', function () { var c = D.tiempoEfectivo[+b.dataset.i]; c[b.dataset.campo] = Math.max(0, num(c[b.dataset.campo]) + num(b.dataset.paso)); guardar(); pintarAnalisis(); }); });
    $$('input[data-campo]', cuerpo).forEach(function (el) { el.addEventListener('change', function () { var c = D.tiempoEfectivo[+el.dataset.i]; c[el.dataset.campo] = el.dataset.campo === 'nombre' ? el.value : Math.max(0, num(el.value)); guardar(); pintarAnalisis(); }); });
    $$('[data-borrar]', cuerpo).forEach(function (b) { b.addEventListener('click', function () { D.tiempoEfectivo.splice(+b.dataset.borrar, 1); guardar(); pintarAnalisis(); }); });
    $('#nuevaCat').addEventListener('click', function () { D.tiempoEfectivo.push({ id: uid('cat'), nombre: 'Nueva categoría', vehiculos: 0, dias: 312, horasDia: 8 }); guardar(); pintarAnalisis(); });
  }
  pintarAnalisis();
};

/* ---------------- VEHÍCULOS ---------------- */
var TIPOS_VEHICULO = {
  'A2': { t: 'A2 · Automóvil / camioneta 2 ejes', llantas: [2, 2, 0], activos: [] },
  'C2': { t: 'C2 · Camión unitario 2 ejes (rabón)', llantas: [2, 4, 0], activos: [] },
  'C3': { t: 'C3 · Camión unitario 3 ejes (tortón)', llantas: [2, 8, 0], activos: [] },
  'T3-S2': { t: 'T3-S2 · Tractocamión articulado 5 ejes', llantas: [2, 8, 8], activos: ['Semirremolque'] },
  'T3-S3': { t: 'T3-S3 · Tractocamión articulado 6 ejes', llantas: [2, 8, 12], activos: ['Semirremolque'] },
  'T3-S2-R4': { t: 'T3-S2-R4 · Doble semirremolque 9 ejes (full)', llantas: [2, 8, 24], activos: ['Semirremolque', 'Remolque y dolly'] },
  'otro': { t: 'Otro (captura libre)', llantas: null, activos: [] }
};
var borradorVehiculo = null;

function perfilVacio() {
  var b = clonar(datosIniciales().vehiculos[0]);
  b.id = uid('veh'); b.nombre = ''; b.descripcion = ''; b.ejemplo = false; b.creado = new Date().toISOString();
  return b;
}

VISTAS.vehiculos = function (cont) {
  var tab = VISTAS.vehiculos.tab || 'listado';
  cont.innerHTML = '<div class="pestanas"><button data-tab="listado">' + ico('clipboard') + 'Listado</button><button data-tab="registro">' + ico('plus') + 'Registro</button></div><div id="vehCuerpo"></div>';
  $$('[data-tab]', cont).forEach(function (b) { b.classList.toggle('activa', b.dataset.tab === tab); b.addEventListener('click', function () { VISTAS.vehiculos.tab = b.dataset.tab; if (b.dataset.tab === 'registro' && !borradorVehiculo) borradorVehiculo = perfilVacio(); VISTAS.vehiculos(cont); }); });
  if (tab === 'registro') { if (!borradorVehiculo) borradorVehiculo = perfilVacio(); formularioVehiculo($('#vehCuerpo')); return; }
  $('#vehCuerpo').innerHTML = '<div class="barra-acciones"><button class="btn primario" id="nuevoPerfil">' + ico('plus') + 'Nuevo perfil</button><div class="con-prefijo" style="width:260px"><span>' + ico('search') + '</span><input type="search" id="busVeh" placeholder="Agregar filtro: buscar…"></div></div><div id="tablaVeh"></div>';
  var t = tablaCatalogo({
    cont: $('#tablaVeh'), filas: function () { return D.vehiculos; },
    columnas: [
      { id: 'nombre', titulo: 'Nombre', render: function (v) { return esc(v.nombre) + (v.ejemplo ? ' <span class="badge ambar" title="Valores de ejemplo: ajústalos a tu unidad real">ejemplo</span>' : ''); } },
      { id: 'descripcion', titulo: 'Descripción' },
      { id: 'tipo', titulo: 'Tipo' },
      { id: 'capacidadLitros', titulo: 'Capacidad (L)', tipo: 'numero', dec: 0, render: function (v) { return v.capacidadLitros ? fmt.ent(v.capacidadLitros) : '—'; } },
      { id: 'km', titulo: 'Costo / km cargado', tipo: 'moneda', valor: function (v) { return Motor.tasasVehiculo(D, v).costoKmCargado; }, render: function (v) { return fmt.mon(Motor.tasasVehiculo(D, v).costoKmCargado); } },
      { id: 'hora', titulo: 'Costo / hora', tipo: 'moneda', valor: function (v) { return Motor.tasasVehiculo(D, v).costoHoraTotal; }, render: function (v) { return fmt.mon(Motor.tasasVehiculo(D, v).costoHoraTotal); } },
      { id: 'creado', titulo: 'Fecha de creación', render: function (v) { return fmt.fechaHora(v.creado); } }
    ],
    alClic: function (v) { borradorVehiculo = clonar(v); VISTAS.vehiculos.tab = 'registro'; VISTAS.vehiculos(cont); },
    acciones: function () {
      return [
        { txt: 'Editar', icono: 'edit', fn: function (v) { borradorVehiculo = clonar(v); VISTAS.vehiculos.tab = 'registro'; VISTAS.vehiculos(cont); } },
        { txt: 'Duplicar', icono: 'copy', fn: function (v) { var n = clonar(v); n.id = uid('veh'); n.nombre = v.nombre + ' (copia)'; n.creado = new Date().toISOString(); D.vehiculos.push(n); guardar(); t.pintar(); } },
        { txt: 'Eliminar', icono: 'trash', peligro: true, fn: async function (v) { if (D.vehiculos.length <= 1) { toast('Debe existir al menos un perfil.', 'warn'); return; } if (await confirmar('¿Eliminar el perfil <b>' + esc(v.nombre) + '</b>? Las cotizaciones guardadas conservan su copia del cálculo.', 'Eliminar', true)) { D.vehiculos = D.vehiculos.filter(function (x) { return x.id !== v.id; }); guardar(); t.pintar(); } } }
      ];
    }
  });
  $('#busVeh').addEventListener('input', function (e) { t.buscar(e.target.value); });
  $('#nuevoPerfil').addEventListener('click', function () { borradorVehiculo = perfilVacio(); VISTAS.vehiculos.tab = 'registro'; VISTAS.vehiculos(cont); });
};

function formularioVehiculo(cont) {
  var v = borradorVehiculo;
  var existe = D.vehiculos.some(function (x) { return x.id === v.id; });
  function inp(ruta, unidad, opt) {
    opt = opt || {};
    var val = getPath(v, ruta);
    return '<div class="campo"><label>' + esc(opt.label) + (unidad ? ' <span class="badge azul" style="padding:0 6px;font-size:10.5px">' + unidad + '</span>' : '') + '</label>' +
      '<input type="' + (opt.texto ? 'text' : 'number') + '" step="any" data-c="' + ruta + '" value="' + esc(val == null ? '' : val) + '"' + (opt.ro ? ' readonly' : '') + (opt.ph ? ' placeholder="' + esc(opt.ph) + '"' : '') + '>' + (opt.ayuda ? '<span class="ayuda">' + esc(opt.ayuda) + '</span>' : '') + '</div>';
  }
  var tipoInfo = TIPOS_VEHICULO[v.tipo] || TIPOS_VEHICULO.otro;
  var llantasFijas = !!tipoInfo.llantas;
  var html =
    '<div class="barra-acciones"><h2 style="margin:0;font-size:17px">' + (existe ? 'Editar perfil' : 'Nuevo perfil') + '</h2><div class="espacio"></div>' +
    '<button class="btn" id="cancelarVeh">Cancelar</button><button class="btn primario" id="guardarVeh">' + ico('save') + 'Guardar perfil</button></div>' +
    (v.ejemplo ? '<div class="aviso" style="margin-bottom:12px">' + ico('alert') + '<div><b>Valores de ejemplo.</b> Este perfil trae datos ilustrativos del manual de referencia; ajústalos a tu unidad real antes de cotizar. <label class="switch" style="margin-left:8px"><input type="checkbox" id="quitarEjemplo"><span class="pista"></span>Ya capturé los datos reales</label></div></div>' : '') +
    '<div class="form-vehiculo"><div>' +
    '<div class="seccion-form"><div class="t">Identificación del vehículo</div><div class="c"><div class="grid-2">' +
    '<div class="campo"><label>Plantilla (copiar valores de…)</label><select id="plantillaVeh"><option value="">— Opcional —</option>' + D.vehiculos.filter(function (x) { return x.id !== v.id; }).map(function (x) { return '<option value="' + x.id + '">' + esc(x.nombre) + '</option>'; }).join('') + '</select><span class="ayuda">Rellena el formulario con otro perfil como punto de partida.</span></div>' +
    inp('nombre', '', { label: 'Nombre del perfil *', texto: true, ph: 'Ej. «Autotanque #01»' }) +
    '<div class="campo"><label>Tipo de vehículo</label><select data-c="tipo">' + Object.keys(TIPOS_VEHICULO).map(function (k) { return '<option value="' + k + '"' + (v.tipo === k ? ' selected' : '') + '>' + TIPOS_VEHICULO[k].t + '</option>'; }).join('') + '</select><span class="ayuda">Pre-llena llantas y activos requeridos; también define la clase para casetas.</span></div>' +
    inp('descripcion', '', { label: 'Descripción', texto: true }) +
    inp('capacidadLitros', 'L', { label: 'Capacidad del autotanque', ayuda: 'Para calcular precio por litro y por m³. Déjalo en 0 si no aplica.' }) +
    '</div></div></div>' +
    '<div class="seccion-form"><div class="t">Dimensiones y peso</div><div class="d">Medidas físicas y peso de la unidad. Sirven para validar la ruta y clasificarla en peajes.</div><div class="c"><div class="grid-3">' +
    inp('dimensiones.alto', 'm', { label: 'Alto' }) + inp('dimensiones.ancho', 'm', { label: 'Ancho' }) + inp('dimensiones.largo', 'm', { label: 'Largo' }) +
    inp('dimensiones.peso', 'ton', { label: 'Peso' }) + inp('dimensiones.capRemolque', 'ton', { label: 'Cap. remolque', ayuda: 'Déjalo en 0 si no aplica.' }) +
    '</div></div></div>' +
    '<div class="seccion-form"><div class="t">Valor del vehículo</div><div class="d">Cuánto vale la unidad y cuánto te durará: con esto se calcula la depreciación y el costo de capital.</div><div class="c"><div class="grid-3">' +
    inp('valor', '$', { label: 'Valor' }) + inp('vidaUtilKm', 'km', { label: 'Vida útil (km)', ayuda: 'Referencia: ~500,000 km camiones, ~600,000 km tractos.' }) + inp('vidaUtilAnios', 'años', { label: 'Vida útil (años)' }) +
    inp('tasaCapitalPct', '% anual', { label: 'Tasa de capital', ayuda: 'Déjala en 0 si el pago del crédito ya está en tus gastos fijos.' }) + inp('valorRescatePct', '%', { label: 'Valor de rescate', ayuda: 'Lo que esperas recuperar al vender la unidad. No se cobra en la tarifa.' }) +
    '</div></div></div>' +
    '<div class="seccion-form"><div class="t">Gestión de neumáticos</div><div class="d">Direccionales (eje delantero), motrices (tracción) y arrastre (remolque). Se calcula el desgaste por kilómetro.' + (llantasFijas ? ' Las cantidades quedan fijas por el tipo de vehículo.' : '') + '</div><div class="c"><div class="grid-3">' +
    ['direccionales', 'motrices', 'arrastre'].map(function (k) {
      var nom = { direccionales: 'Direccionales', motrices: 'Motrices', arrastre: 'Arrastre' }[k];
      var sinArrastre = k === 'arrastre' && num(v.neumaticos.arrastre.cantidad) === 0;
      return inp('neumaticos.' + k + '.cantidad', '', { label: nom + ' — Cantidad', ro: llantasFijas }) + inp('neumaticos.' + k + '.vidaKm', 'km', { label: nom + ' — Vida útil', ro: sinArrastre }) + inp('neumaticos.' + k + '.precio', '$', { label: nom + ' — Precio / unidad', ro: sinArrastre });
    }).join('') +
    '</div></div></div>' +
    '<div class="seccion-form"><div class="t">Combustible</div><div class="d">Rendimiento y costo de combustible: uno de los rubros que más pesa en cada viaje.</div><div class="c"><div class="grid-3">' +
    inp('combustible.kmLVacio', 'km/L', { label: 'Km/L vacío' }) + inp('combustible.kmLCargado', 'km/L', { label: 'Km/L cargado', ayuda: 'Llena el tanque, recorre 500–1,000 km cargado, vuelve a llenar y divide km entre litros.' }) + inp('combustible.precioL', '$/L', { label: 'Precio / L' }) +
    inp('combustible.tanqueL', 'L', { label: 'Tanque' }) + inp('combustible.recargaMin', 'min', { label: 'Recarga', ayuda: 'Minutos para llenar el tanque.' }) + inp('combustible.ralentiLh', 'L/h', { label: 'Ralentí', ayuda: 'Opcional. Referencia: ~3 L/h tracto, ~1.5–2 L/h camión.' }) +
    '</div></div></div>' +
    '<div class="seccion-form"><div class="t">Descanso del operador</div><div class="d">Reglas de descanso del chofer en ruta: generan paradas que suman tiempo y un costo por cada una.</div><div class="c"><div class="grid-3">' +
    inp('descanso.frecuenciaH', 'hrs', { label: 'Frecuencia', ayuda: 'Cada cuántas horas de manejo descansa.' }) + inp('descanso.duracionH', 'hrs', { label: 'Duración descanso' }) + inp('descanso.costoParada', '$', { label: 'Costo por parada', ayuda: 'Alimentos, pernocta…' }) +
    '</div></div></div>' +
    '<div class="seccion-form"><div class="t">Mantenimiento</div><div class="d">Cada cuánto y cuánto cuesta el mantenimiento preventivo. Se reparte por kilómetro.</div><div class="c"><div class="grid-2">' +
    inp('mantenimiento.intervaloKm', 'km', { label: 'Intervalo', ayuda: 'Referencia: ~20,000 km camiones, ~25,000 km tractos.' }) + inp('mantenimiento.costo', '$', { label: 'Costo mtto', ayuda: 'Tip: suma tus facturas del último año y divide entre los km recorridos.' }) +
    '</div></div></div>' +
    '<div class="seccion-form"><div class="t">Activos requeridos</div><div class="d">Remolques, cajas, tanques u otros activos que la unidad necesita. Se deprecian igual que el vehículo.</div><div class="c"><div class="lista-mini" id="listaActivos">' +
    v.activos.map(function (a, i) {
      return '<div class="item" style="grid-template-columns:1.4fr 1fr 1fr .8fr auto">' +
        '<div class="campo"><label>Activo</label><input type="text" data-lista="activos" data-i="' + i + '" data-k="nombre" value="' + esc(a.nombre) + '"></div>' +
        '<div class="campo"><label>Valor $</label><input type="number" step="any" data-lista="activos" data-i="' + i + '" data-k="valor" value="' + esc(a.valor) + '"></div>' +
        '<div class="campo"><label>Vida útil (km)</label><input type="number" step="any" data-lista="activos" data-i="' + i + '" data-k="vidaKm" value="' + esc(a.vidaKm) + '"></div>' +
        '<div class="campo"><label>Vida (años)</label><input type="number" step="any" data-lista="activos" data-i="' + i + '" data-k="vidaAnios" value="' + esc(a.vidaAnios) + '"></div>' +
        '<button class="btn icono fantasma peligro" data-quitar-lista="activos" data-i="' + i + '" title="Quitar">' + ico('trash') + '</button></div>';
    }).join('') + '</div><button class="btn chico" style="margin-top:8px" data-agregar="activos">' + ico('plus') + 'Agregar activo</button></div></div>' +
    '<div class="seccion-form"><div class="t">Gastos fijos del vehículo</div><div class="d">Elige la categoría (define cuántas horas al año trabaja) y agrega los gastos propios de la unidad: seguro, tenencia, GPS…</div><div class="c">' +
    '<div class="campo" style="max-width:320px;margin-bottom:10px"><label>Categoría de vehículo</label><select data-c="categoriaId"><option value="">— Sin categoría —</option>' + D.tiempoEfectivo.map(function (c) { return '<option value="' + c.id + '"' + (v.categoriaId === c.id ? ' selected' : '') + '>' + esc(c.nombre) + ' · ' + fmt.ent(c.dias * c.horasDia) + ' h/año</option>'; }).join('') + '</select><span class="ayuda">Sin categoría, los gastos fijos y el equipamiento no se pueden repartir y quedan en cero.</span></div>' +
    '<div class="lista-mini">' + v.gastosFijos.map(function (g, i) {
      return '<div class="item" style="grid-template-columns:1.6fr 1fr 1fr auto">' +
        '<div class="campo"><label>Costo fijo directo — Nombre</label><input type="text" data-lista="gastosFijos" data-i="' + i + '" data-k="nombre" value="' + esc(g.nombre) + '"></div>' +
        '<div class="campo"><label>Monto $</label><input type="number" step="any" data-lista="gastosFijos" data-i="' + i + '" data-k="monto" value="' + esc(g.monto) + '"></div>' +
        '<div class="campo"><label>Periodo de pago</label><select data-lista="gastosFijos" data-i="' + i + '" data-k="periodo">' + OPC_PERIODO.map(function (o) { return '<option value="' + o.v + '"' + (g.periodo === o.v ? ' selected' : '') + '>' + o.t + '</option>'; }).join('') + '</select></div>' +
        '<button class="btn icono fantasma peligro" data-quitar-lista="gastosFijos" data-i="' + i + '" title="Quitar">' + ico('trash') + '</button></div>';
    }).join('') + '</div><button class="btn chico" style="margin-top:8px" data-agregar="gastosFijos">' + ico('plus') + 'Agregar gasto</button></div></div>' +
    '<div class="seccion-form"><div class="t">Equipamiento de la unidad</div><div class="d">Equipo de abordo del catálogo de tu empresa. Su costo se reparte según cada cuánto lo reemplazas.</div><div class="c"><div class="lista-mini">' +
    v.equipamiento.map(function (e, i) {
      var eq = D.equipamiento.find(function (x) { return x.id === e.equipoId; });
      return '<div class="item" style="grid-template-columns:2fr .7fr .7fr .7fr auto">' +
        '<div class="campo"><label>Equipo' + (eq ? ' · ' + fmt.mon(eq.precio) : '') + '</label><select data-lista="equipamiento" data-i="' + i + '" data-k="equipoId"><option value="">— Selecciona —</option>' + D.equipamiento.map(function (x) { return '<option value="' + x.id + '"' + (x.id === e.equipoId ? ' selected' : '') + '>' + esc(x.equipo) + '</option>'; }).join('') + '</select></div>' +
        '<div class="campo"><label>Cantidad</label><input type="number" min="1" data-lista="equipamiento" data-i="' + i + '" data-k="cantidad" value="' + esc(e.cantidad) + '"></div>' +
        '<div class="campo"><label>Cambio (años)</label><input type="number" min="0" data-lista="equipamiento" data-i="' + i + '" data-k="anios" value="' + esc(e.anios) + '"></div>' +
        '<div class="campo"><label>+ meses</label><input type="number" min="0" max="11" data-lista="equipamiento" data-i="' + i + '" data-k="meses" value="' + esc(e.meses) + '"></div>' +
        '<button class="btn icono fantasma peligro" data-quitar-lista="equipamiento" data-i="' + i + '" title="Quitar">' + ico('trash') + '</button></div>';
    }).join('') + '</div><button class="btn chico" style="margin-top:8px" data-agregar="equipamiento">' + ico('plus') + 'Agregar equipo</button></div></div>' +
    '<div class="seccion-form"><div class="t">Pago del operador (valores de partida)</div><div class="d">Lo que se le paga al operador por cada viaje. Se puede ajustar al cotizar. El sueldo base va en gastos fijos.</div><div class="c"><div class="grid-3">' +
    inp('pagoOperador.operadores', '', { label: 'Operadores' }) + inp('pagoOperador.kmCargado', '$/km', { label: 'Por km cargado' }) + inp('pagoOperador.kmVacio', '$/km', { label: 'Por km vacío' }) +
    inp('pagoOperador.horaCargado', '$/h', { label: 'Por hora con carga' }) + inp('pagoOperador.horaVacio', '$/h', { label: 'Por hora sin carga' }) +
    '<div class="campo"><label>Esquema</label><div class="pila" style="gap:6px;padding-top:4px">' +
    '<label class="switch"><input type="checkbox" data-c="pagoOperador.porKm"' + (v.pagoOperador.porKm ? ' checked' : '') + '><span class="pista"></span>Por kilómetro</label>' +
    '<label class="switch"><input type="checkbox" data-c="pagoOperador.porHora"' + (v.pagoOperador.porHora ? ' checked' : '') + '><span class="pista"></span>Por hora</label></div></div>' +
    '</div></div></div>' +
    '</div><div class="impacto" id="impactoVeh"></div></div>';
  cont.innerHTML = '<div class="fv-raiz">' + html + '</div>';
  var raiz = cont.firstChild;

  function pintarImpacto() {
    var t = Motor.tasasVehiculo(D, v);
    var hd = t.categoria ? t.categoria.horasDia : 0;
    $('#impactoVeh').innerHTML =
      '<div class="bloque"><div class="t">' + ico('chart') + 'Impacto del perfil</div>' +
      '<div class="r"><span>Combustible (cargado)</span><b>' + fmt.mon(t.combKmCargado) + '/km</b></div>' +
      '<div class="r"><span>Neumáticos</span><b>' + fmt.mon(t.neum.total) + '/km</b></div>' +
      '<div class="r"><span>Mantenimiento</span><b>' + fmt.mon(t.mttoKm) + '/km</b></div>' +
      '<div class="r"><span>Depreciación</span><b>' + fmt.mon(t.depKm) + '/km</b></div>' +
      '<div class="r tot"><span>Costo por km cargado</span><b>' + fmt.mon(t.costoKmCargado) + '</b></div>' +
      '<div class="r"><span>Gastos fijos generales</span><b>' + fmt.mon(t.gfGenHora) + '/h</b></div>' +
      '<div class="r"><span>Gastos fijos específicos</span><b>' + fmt.mon(t.gfEspHora) + '/h</b></div>' +
      '<div class="r"><span>Equipamiento<small>' + fmt.mon(t.equipDia) + '/día</small></span><b>' + fmt.mon(t.equipHora) + '/h</b></div>' +
      '<div class="r"><span>Costo de capital<small>' + fmt.num(t.tasaCapitalPct, 1) + '% s/ ' + fmt.mon(t.capitalPromedio, 0) + '</small></span><b>' + fmt.mon(t.capitalHora) + '/h</b></div>' +
      '<div class="r tot"><span>Costo por hora de servicio</span><b>' + fmt.mon(t.costoHoraTotal) + '</b></div>' +
      '<div class="r"><span>Costo fijo por día (' + hd + ' h)</span><b>' + fmt.mon(t.costoHoraTotal * hd) + '</b></div>' +
      (t.categoria ? '' : '<div class="pie-nota" style="color:var(--warn)">' + ico('alert') + ' Sin categoría: gastos fijos y equipamiento en cero.</div>') +
      '<div class="pie-nota">No incluye peajes, pago al operador ni descansos: esos dependen de cada viaje.</div></div>';
  }
  pintarImpacto();

  raiz.addEventListener('input', function (e) {
    var el = e.target;
    if (el.dataset.c && el.type !== 'checkbox' && el.tagName !== 'SELECT') { setPath(v, el.dataset.c, el.type === 'number' ? (el.value === '' ? '' : num(el.value)) : el.value); el.classList.remove('invalido'); pintarImpacto(); }
    if (el.dataset.lista && el.tagName !== 'SELECT') { v[el.dataset.lista][+el.dataset.i][el.dataset.k] = el.type === 'number' ? num(el.value) : el.value; pintarImpacto(); }
  });
  raiz.addEventListener('change', function (e) {
    var el = e.target;
    if (el.id === 'plantillaVeh' && el.value) {
      var src = clonar(D.vehiculos.find(function (x) { return x.id === el.value; }));
      src.id = v.id; src.nombre = v.nombre || ''; src.creado = v.creado; borradorVehiculo = src; src.ejemplo = !!src.ejemplo; formularioVehiculo(cont); toast('Valores copiados. Ajusta lo necesario y guarda.'); return;
    }
    if (el.id === 'quitarEjemplo') { v.ejemplo = !el.checked; return; }
    if (el.dataset.c === 'tipo') {
      v.tipo = el.value; var info = TIPOS_VEHICULO[v.tipo];
      if (info && info.llantas) {
        v.neumaticos.direccionales.cantidad = info.llantas[0]; v.neumaticos.motrices.cantidad = info.llantas[1]; v.neumaticos.arrastre.cantidad = info.llantas[2];
        info.activos.forEach(function (nombre, i) { if (!v.activos[i]) v.activos.push({ id: uid('ac'), nombre: nombre, valor: 0, vidaKm: 1000000, vidaAnios: 15 }); });
        if (!info.activos.length && v.activos.length) toast('Este tipo no requiere activos adicionales; revisa la lista de activos.', 'warn');
      }
      formularioVehiculo(cont); return;
    }
    if (el.dataset.c && (el.type === 'checkbox' || el.tagName === 'SELECT')) { setPath(v, el.dataset.c, el.type === 'checkbox' ? el.checked : el.value); pintarImpacto(); return; }
    if (el.dataset.lista && el.tagName === 'SELECT') { v[el.dataset.lista][+el.dataset.i][el.dataset.k] = el.value; formularioVehiculo(cont); }
  });
  raiz.addEventListener('click', function (e) {
    var ag = e.target.closest('[data-agregar]');
    if (ag) {
      var k = ag.dataset.agregar;
      if (k === 'activos') v.activos.push({ id: uid('ac'), nombre: 'Activo', valor: 0, vidaKm: 1000000, vidaAnios: 15 });
      if (k === 'gastosFijos') v.gastosFijos.push({ id: uid('gv'), nombre: 'Gasto', monto: 0, periodo: 'mensual' });
      if (k === 'equipamiento') v.equipamiento.push({ id: uid('ve'), equipoId: '', cantidad: 1, anios: 1, meses: 0 });
      formularioVehiculo(cont); return;
    }
    var q = e.target.closest('[data-quitar-lista]');
    if (q) { v[q.dataset.quitarLista].splice(+q.dataset.i, 1); formularioVehiculo(cont); return; }
    if (e.target.closest('#cancelarVeh')) { borradorVehiculo = null; VISTAS.vehiculos.tab = 'listado'; navegar(); return; }
    if (e.target.closest('#guardarVeh')) guardarVehiculo();
  });

  function guardarVehiculo() {
    var obligatorios = ['dimensiones.alto', 'dimensiones.ancho', 'dimensiones.largo', 'dimensiones.peso', 'valor', 'vidaUtilKm', 'vidaUtilAnios',
      'neumaticos.direccionales.cantidad', 'neumaticos.direccionales.vidaKm', 'neumaticos.direccionales.precio', 'neumaticos.motrices.cantidad', 'neumaticos.motrices.vidaKm', 'neumaticos.motrices.precio',
      'combustible.kmLVacio', 'combustible.kmLCargado', 'combustible.precioL', 'combustible.tanqueL', 'combustible.recargaMin',
      'descanso.frecuenciaH', 'descanso.duracionH', 'mantenimiento.intervaloKm', 'mantenimiento.costo'];
    var errores = [];
    if (!String(v.nombre || '').trim()) { errores.push('nombre'); }
    obligatorios.forEach(function (r) { if (!(num(getPath(v, r)) > 0)) errores.push(r); });
    if (num(v.neumaticos.arrastre.cantidad) > 0 && (!(num(v.neumaticos.arrastre.vidaKm) > 0) || !(num(v.neumaticos.arrastre.precio) > 0))) errores.push('neumaticos.arrastre.vidaKm');
    v.equipamiento.forEach(function (e) { if (num(e.anios) + num(e.meses) <= 0) errores.push('equipamiento'); });
    $$('[data-c]', cont).forEach(function (el) { el.classList.toggle('invalido', errores.indexOf(el.dataset.c) >= 0); });
    if (errores.length) { toast('Revisa los campos marcados: son obligatorios y deben ser mayores a 0.' + (errores.indexOf('equipamiento') >= 0 ? ' En equipamiento, años + meses no puede ser 0.' : ''), 'warn', 6000); var p = $('.invalido', cont); if (p) p.scrollIntoView({ block: 'center' }); return; }
    if (!v.categoriaId) toast('Guardado sin categoría: gastos fijos y equipamiento quedarán en cero al cotizar.', 'warn', 6000);
    v.equipamiento = v.equipamiento.filter(function (e) { return e.equipoId; });
    var i = D.vehiculos.findIndex(function (x) { return x.id === v.id; });
    if (i >= 0) D.vehiculos[i] = clonar(v); else D.vehiculos.push(clonar(v));
    guardar(); borradorVehiculo = null; VISTAS.vehiculos.tab = 'listado'; toast('Perfil <b>' + esc(v.nombre) + '</b> guardado.', 'ok'); navegar();
  }
}

/* ============================================================
   COTIZAR SERVICIO: flete por ruta y renta
   ============================================================ */
var Directorio = { estaciones: [], terminales: [], cargado: false, intentado: false, error: '' };
var mapa = null, capaRuta = null, capaMarcas = null;
var Cot = null;

function cargarDirectorioTexto(texto) {
  var filas = parseCSV(texto);
  if (filas.length < 2) throw new Error('El CSV no tiene datos.');
  var h = filas[0].map(function (x) { return x.replace(/^﻿/, '').trim(); });
  var iLat = h.indexOf('pv.Latitud_Y'), iLon = h.indexOf('pv.Longitud_X'), iPer = h.indexOf('pv.Permiso');
  var iTLat = h.indexOf('r.LATITUD'), iTLon = h.indexOf('r.LONGUITUD'), iTad = h.indexOf('r.TAD');
  if ([iLat, iLon, iPer, iTLat, iTLon, iTad].indexOf(-1) >= 0) throw new Error('Faltan columnas: pv.Permiso, pv.Latitud_Y, pv.Longitud_X, r.TAD, r.LATITUD, r.LONGUITUD.');
  var est = {}, ter = {};
  for (var i = 1; i < filas.length; i++) {
    var f = filas[i];
    var la = parseFloat(f[iLat]), lo = parseFloat(f[iLon]);
    if (f[iPer] && isFinite(la) && isFinite(lo)) est[f[iPer]] = { nombre: f[iPer], lat: la, lon: lo };
    var tla = parseFloat(f[iTLat]), tlo = parseFloat(f[iTLon]);
    if (f[iTad] && isFinite(tla) && isFinite(tlo)) ter[f[iTad]] = { nombre: f[iTad], lat: tla, lon: tlo };
  }
  Directorio.estaciones = Object.values(est).sort(function (a, b) { return a.nombre.localeCompare(b.nombre); });
  Directorio.terminales = Object.values(ter).sort(function (a, b) { return a.nombre.localeCompare(b.nombre); });
  Directorio.cargado = true; Directorio.error = '';
}
async function cargarDirectorio() {
  if (Directorio.intentado) return; Directorio.intentado = true;
  try {
    var r = await fetch(encodeURI(D.parametros.directorioCsv), { cache: 'no-store' });
    if (!r.ok) throw new Error('HTTP ' + r.status);
    cargarDirectorioTexto(await r.text());
  } catch (e) {
    Directorio.error = 'No se encontró «' + D.parametros.directorioCsv + '» junto al HTML (' + e.message + '). Cárgalo manualmente en Configuración o busca lugares en el mapa.';
  }
}

function nuevoCot(vehiculoId) {
  var v = vehiculoPorId(vehiculoId) || D.vehiculos[0];
  return {
    modo: 'flete', vehiculoId: v.id, clienteId: '', trafico: 'con', vmax: num(D.parametros.velMaxDefault) || 90,
    paradas: [nuevaParada(true), nuevaParada(false)],
    ruta: null, calculando: false, seq: 0, peajesManual: null,
    ajustes: ajustesDeVehiculo(v), vacios: Motor.vaciosPorDefecto(), otros: [],
    tipoCargaId: (D.tiposCarga[0] || {}).id || '', aumentoPct: num((D.tiposCarga[0] || {}).aumentoPct), toneladas: 0,
    utilidadPct: num(D.parametros.utilidadPct), financiamientoPct: num(D.parametros.financiamientoPct), ivaPct: num(D.parametros.ivaPct),
    renta: { unidad: 'mensual', cantidad: 1, operadores: 1, sueldoOperadorMensual: 8000, kmPeriodo: 0, combustibleIncluido: false, casetasIncluidas: false, casetasPeriodo: 0, topeKm: 0, recargoExcedentePct: 50, precioFijoMensual: '', viajes: 1, frecuencia: 'dia', utilidadPct: num(D.parametros.utilidadPct), ivaPct: num(D.parametros.ivaPct) },
    detalleAbierto: false, detalleTab: 'detallado', res: null, cotizacionDestinoId: null
  };
}
function nuevaParada(conCarga) { return { id: uid('p'), texto: '', lat: null, lon: null, conCarga: conCarga !== false, minParada: 0, minRalenti: 0, origen: '' }; }
function ajustesDeVehiculo(v) {
  return { precioL: null, kmLCargado: null, kmLVacio: null, costoDescanso: null, operadores: num(v.pagoOperador.operadores) || 1, pago: clonar(v.pagoOperador), pagoConUtilidad: true };
}
function vehiculoCot() { return vehiculoPorId(Cot.vehiculoId) || D.vehiculos[0]; }
function paradasValidas() { return Cot.paradas.filter(function (p) { return isFinite(p.lat) && isFinite(p.lon) && p.lat !== null && p.lon !== null; }); }

/* ---------- Mapa ---------- */
function iniciarMapa() {
  if (mapa) { setTimeout(function () { mapa.invalidateSize(); }, 50); return; }
  if (typeof L === 'undefined') { $('#mapa').innerHTML = '<div class="aviso error" style="margin:80px auto;max-width:420px">' + ico('alert') + '<div>No se pudo cargar la librería de mapas (Leaflet). Revisa tu conexión a internet.</div></div>'; return; }
  mapa = L.map('mapa', { zoomControl: false }).setView([21.5, -97.5], 6);
  L.control.zoom({ position: 'topright' }).addTo(mapa);
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { attribution: '© OpenStreetMap', maxZoom: 19, crossOrigin: 'anonymous' }).addTo(mapa);
  capaRuta = L.layerGroup().addTo(mapa);
  capaMarcas = L.layerGroup().addTo(mapa);
  mapa.on('click', function (e) {
    if (Cot.eligiendo) {
      var p = Cot.paradas.find(function (x) { return x.id === Cot.eligiendo; });
      if (p) { p.lat = e.latlng.lat; p.lon = e.latlng.lng; p.texto = 'Punto en mapa (' + e.latlng.lat.toFixed(5) + ', ' + e.latlng.lng.toFixed(5) + ')'; p.origen = 'mapa'; }
      Cot.eligiendo = null; $('#vistaCotizar').classList.remove('cursor-mapa');
      pintarPanelViaje(); calcularRutaPronto(); return;
    }
    var pop = L.popup().setLatLng(e.latlng).setContent('<button class="btn chico primario" id="agregarAqui">' + ico('pin') + 'Agregar parada aquí</button>').openOn(mapa);
    setTimeout(function () {
      var b = document.getElementById('agregarAqui'); if (!b) return;
      b.addEventListener('click', function () {
        var vacia = Cot.paradas.find(function (x) { return x.lat == null; });
        var p = vacia || nuevaParada(true);
        if (!vacia) Cot.paradas.push(p);
        p.lat = e.latlng.lat; p.lon = e.latlng.lng; p.texto = 'Punto en mapa (' + p.lat.toFixed(5) + ', ' + p.lon.toFixed(5) + ')'; p.origen = 'mapa';
        mapa.closePopup(pop); pintarPanelViaje(); calcularRutaPronto();
      });
    }, 0);
  });
  setTimeout(function () { mapa.invalidateSize(); }, 80);
}

function pintarMapa() {
  if (!mapa) return;
  capaRuta.clearLayers(); capaMarcas.clearLayers();
  var pv = Cot.paradas;
  var ultimo = -1; pv.forEach(function (p, i) { if (p.lat != null) ultimo = i; });
  var n = 0;
  pv.forEach(function (p, i) {
    if (p.lat == null) return; n++;
    var icono = L.divIcon({ className: '', html: '<div class="marcador-num' + (i === ultimo && n > 1 ? ' fin' : '') + '"><span>' + n + '</span></div>', iconSize: [26, 26], iconAnchor: [13, 26] });
    L.marker([p.lat, p.lon], { icon: icono, title: p.texto }).bindTooltip(esc(p.texto || 'Parada ' + n)).addTo(capaMarcas);
  });
  var limites = [];
  if (Cot.ruta) {
    Cot.ruta.tramos.forEach(function (t) {
      if (!t.coords || !t.coords.length) return;
      L.polyline(t.coords, t.conCarga ? { color: '#4a9eff', weight: 5, opacity: .95 } : { color: '#9aa0a8', weight: 4, opacity: .9, dashArray: '8 8' })
        .bindTooltip((t.conCarga ? 'Con carga' : 'Sin carga') + ' · ' + fmt.km(t.km) + ' · ' + fmt.dur(t.horas)).addTo(capaRuta);
      limites = limites.concat(t.coords);
    });
    (Cot.ruta.casetas || []).forEach(function (c) {
      L.circleMarker([c.lat, c.lon], { radius: 6, color: '#fff', weight: 2, fillColor: '#f2b01e', fillOpacity: 1 }).bindTooltip('Caseta: ' + esc(c.nombre) + ' · ' + fmt.mon(c.tarifa)).addTo(capaMarcas);
    });
  }
  if (!limites.length) limites = paradasValidas().map(function (p) { return [p.lat, p.lon]; });
  if (limites.length > 1) mapa.fitBounds(L.latLngBounds(limites), { padding: [40, 40] });
  else if (limites.length === 1) mapa.setView(limites[0], 11);
}

/* ---------- Ruteo ---------- */
function distKm(a, b) {
  var R = 6371, dLa = (b[0] - a[0]) * Math.PI / 180, dLo = (b[1] - a[1]) * Math.PI / 180;
  var h = Math.sin(dLa / 2) * Math.sin(dLa / 2) + Math.cos(a[0] * Math.PI / 180) * Math.cos(b[0] * Math.PI / 180) * Math.sin(dLo / 2) * Math.sin(dLo / 2);
  return 2 * R * Math.atan2(Math.sqrt(h), Math.sqrt(1 - h));
}
function distPuntoSegmentoKm(p, a, b) {
  var k = 111.32, kl = 111.32 * Math.cos(p[0] * Math.PI / 180);
  var px = p[1] * kl, py = p[0] * k, ax = a[1] * kl, ay = a[0] * k, bx = b[1] * kl, by = b[0] * k;
  var dx = bx - ax, dy = by - ay;
  var t = dx === 0 && dy === 0 ? 0 : Math.max(0, Math.min(1, ((px - ax) * dx + (py - ay) * dy) / (dx * dx + dy * dy)));
  return Math.hypot(px - (ax + t * dx), py - (ay + t * dy));
}
function cruzaZona(coords, z) {
  if (z.tramo && z.tramo.length > 1) {
    for (var i = 0; i < coords.length; i++) for (var j = 0; j < z.tramo.length - 1; j++) if (distPuntoSegmentoKm(coords[i], z.tramo[j], z.tramo[j + 1]) <= num(z.umbralKm || 1.2)) return true;
    return false;
  }
  for (var k = 0; k < coords.length; k++) if (distKm(coords[k], [z.lat, z.lon]) <= num(z.radioKm)) return true;
  return false;
}
function simplificar(coords, max) {
  if (!coords || coords.length <= max) return coords;
  var paso = coords.length / max, r = [];
  for (var i = 0; i < max; i++) r.push(coords[Math.floor(i * paso)]);
  r.push(coords[coords.length - 1]);
  return r.map(function (c) { return [Math.round(c[0] * 1e5) / 1e5, Math.round(c[1] * 1e5) / 1e5]; });
}

async function pedirRuta(puntos) {
  var cfg = D.parametros.ruteo || {};
  var ctrl = new AbortController(); var tt = setTimeout(function () { ctrl.abort(); }, 20000);
  try {
    if (cfg.proveedor === 'ors' && cfg.orsApiKey) {
      var r = await fetch('https://api.openrouteservice.org/v2/directions/driving-hgv/geojson', {
        method: 'POST', signal: ctrl.signal,
        headers: { 'Authorization': cfg.orsApiKey, 'Content-Type': 'application/json' },
        body: JSON.stringify({ coordinates: puntos.map(function (p) { return [p[1], p[0]]; }) })
      });
      if (!r.ok) throw new Error('OpenRouteService respondió ' + r.status);
      var j = await r.json(); var f = j.features && j.features[0];
      if (!f) throw new Error('Sin ruta');
      return { km: f.properties.summary.distance / 1000, horas: f.properties.summary.duration / 3600, coords: f.geometry.coordinates.map(function (c) { return [c[1], c[0]]; }), motor: 'OpenRouteService (camión)' };
    }
    var url = 'https://router.project-osrm.org/route/v1/driving/' + puntos.map(function (p) { return p[1] + ',' + p[0]; }).join(';') + '?overview=full&geometries=geojson';
    var r2 = await fetch(url, { signal: ctrl.signal });
    if (!r2.ok) throw new Error('OSRM respondió ' + r2.status);
    var j2 = await r2.json();
    if (!j2.routes || !j2.routes.length) throw new Error('Sin ruta');
    return { km: j2.routes[0].distance / 1000, horas: j2.routes[0].duration / 3600, coords: j2.routes[0].geometry.coordinates.map(function (c) { return [c[1], c[0]]; }), motor: 'OSRM (perfil automóvil)' };
  } finally { clearTimeout(tt); }
}

async function rutaTramo(a, b, avisos) {
  var directa = await pedirRuta([a, b]);
  var zonas = (D.zonasRestringidas || []).filter(function (z) {
    var dentro = function (p) { return z.tramo ? false : distKm(p, [z.lat, z.lon]) <= num(z.radioKm); };
    return !dentro(a) && !dentro(b) && cruzaZona(directa.coords, z);
  });
  if (!zonas.length) return directa;
  var z = zonas[0], mejor = null;
  for (var i = 0; i < (z.alternas || []).length; i++) {
    try {
      var alt = await pedirRuta([a].concat(z.alternas[i].puntos).concat([b]));
      if (!mejor || alt.km < mejor.km) { mejor = alt; mejor.nombreAlterna = z.alternas[i].nombre; }
    } catch (e) { /* siguiente alterna */ }
  }
  if (!mejor) { avisos.push({ tipo: 'error', txt: 'La ruta cruza «' + z.nombre + '» y no se encontró una alterna disponible. Revisa el trazado.' }); return directa; }
  avisos.push({ tipo: 'info', txt: 'Se evitó «' + z.nombre + '» con la ruta obligada por ' + mejor.nombreAlterna + '.' });
  return mejor;
}

var calcularRutaPronto = debounce(function () { calcularRuta(); }, 450);

async function calcularRuta() {
  var pv = paradasValidas();
  var seq = ++Cot.seq;
  if (pv.length < 2) { Cot.ruta = null; Cot.calculando = false; pintarMapa(); recalcular(); return; }
  Cot.calculando = true; pintarResultadosRuta();
  var avisos = [], tramos = [];
  try {
    for (var i = 0; i < pv.length - 1; i++) {
      var a = [pv[i].lat, pv[i].lon], b = [pv[i + 1].lat, pv[i + 1].lon];
      var t = await rutaTramo(a, b, avisos);
      if (seq !== Cot.seq) return;
      tramos.push({ desde: pv[i].texto, hasta: pv[i + 1].texto, km: t.km, horasBase: t.horas, coords: simplificar(t.coords, 600), conCarga: !!pv[i].conCarga, motor: t.motor, alterna: t.nombreAlterna || '' });
    }
  } catch (e) {
    if (seq !== Cot.seq) return;
    Cot.calculando = false; Cot.ruta = null;
    toast('No se pudo trazar la ruta: ' + esc(e.message === 'The user aborted a request.' ? 'tiempo de espera agotado' : e.message) + '. Revisa tu conexión o el proveedor de ruteo en Configuración.', 'error', 7000);
    pintarResultadosRuta(); pintarMapa(); return;
  }
  Cot.calculando = false;
  Cot.ruta = { tramos: tramos, avisos: avisos, motor: tramos[0] ? tramos[0].motor : '' };
  ajustarTiempos(); detectarCasetas();
  pintarMapa(); recalcular(); pintarPanelViaje();
}

function ajustarTiempos() {
  if (!Cot.ruta) return;
  var factor = Cot.trafico === 'con' ? (num(D.parametros.factorTrafico) || 1) : 1;
  var vmax = num(Cot.vmax);
  Cot.ruta.tramos.forEach(function (t) {
    var h = num(t.horasBase) * factor;
    if (vmax > 0) h = Math.max(h, t.km / vmax);
    t.horas = h;
  });
}

function claseVehiculo(v) { return v.tipo && v.tipo !== 'otro' ? v.tipo : 'C2'; }
function detectarCasetas(ruta, v) {
  ruta = ruta || Cot.ruta; v = v || vehiculoCot();
  if (!ruta) return 0;
  var clase = claseVehiculo(v), lista = [];
  (D.casetas || []).forEach(function (c) {
    if (!isFinite(c.lat) || !isFinite(c.lon)) return;
    ruta.tramos.forEach(function (t) {
      var cerca = (t.coords || []).some(function (p, i, arr) { return i > 0 && distPuntoSegmentoKm([c.lat, c.lon], arr[i - 1], p) <= 0.35; });
      if (cerca) lista.push({ id: c.id, nombre: c.nombre, lat: c.lat, lon: c.lon, tarifa: num((c.tarifas || {})[clase]) });
    });
  });
  ruta.casetas = lista;
  return lista.reduce(function (a, c) { return a + c.tarifa; }, 0);
}
function peajesRuta() {
  if (Cot.peajesManual != null && Cot.peajesManual !== '') return num(Cot.peajesManual);
  return Cot.ruta ? (Cot.ruta.casetas || []).reduce(function (a, c) { return a + c.tarifa; }, 0) : 0;
}

/* ---------- Costeo ---------- */
function entradaFlete() {
  var v = vehiculoCot(), cli = clientePorId(Cot.clienteId);
  return {
    tramos: Cot.ruta ? Cot.ruta.tramos.map(function (t) { return { km: t.km, horas: t.horas, conCarga: t.conCarga }; }) : [],
    paradas: Cot.paradas.map(function (p) { return { minParada: p.minParada, minRalenti: p.minRalenti }; }),
    peajes: peajesRuta(), ajustes: Cot.ajustes, vacios: Cot.vacios, otrosGastos: Cot.otros,
    aumentoPct: Cot.aumentoPct, toneladas: Cot.toneladas, utilidadPct: Cot.utilidadPct, financiamientoPct: Cot.financiamientoPct, ivaPct: Cot.ivaPct,
    reglaInterna: cli && cli.tipo === 'grupo' ? D.parametros.reglaGrupo : null
  };
}
function recalcular() {
  var v = vehiculoCot();
  if (Cot.modo === 'renta') { Cot.resRenta = Motor.costearRenta(D, v, Object.assign({ ajustes: Cot.ajustes }, Cot.renta)); pintarResultadoRenta(); return; }
  Cot.res = Cot.ruta ? Motor.costearViaje(D, v, entradaFlete()) : null;
  pintarResultadosRuta();
  if (Cot.detalleAbierto) actualizarSalidasDetalle();
}

/* ---------- Panel "Nuevo viaje" ---------- */
VISTAS.cotizar = function () {
  if (!Cot) Cot = nuevoCot((D.vehiculos[0] || {}).id);
  if (!vehiculoPorId(Cot.vehiculoId)) { Cot.vehiculoId = D.vehiculos[0].id; Cot.ajustes = ajustesDeVehiculo(D.vehiculos[0]); }
  iniciarMapa();
  $('#btnPrimerosPasos').innerHTML = ico('book') + 'Primeros pasos';
  $('#btnPrimerosPasos').onclick = function () { location.hash = '#/documentacion'; setTimeout(function () { var s = document.getElementById('doc-flujo'); if (s) s.scrollIntoView(); }, 60); };
  $('#btnMinPanel').innerHTML = ico('minimize');
  $('#btnMinPanel').onclick = function () { $('#panelViaje').classList.toggle('minimizado'); };
  cargarDirectorio().then(function () { if (vistaActual === 'cotizar') pintarPanelViaje(); });
  pintarPanelViaje(); pintarMapa(); recalcular();
  if (Cot.detalleAbierto) pintarDetalle();
};

function opcionesVehiculos(sel) { return D.vehiculos.map(function (v) { return '<option value="' + v.id + '"' + (v.id === sel ? ' selected' : '') + '>' + esc(v.nombre) + (v.ejemplo ? ' · ejemplo' : '') + '</option>'; }).join(''); }

function pintarPanelViaje() {
  var c = $('#panelViajeCuerpo'); if (!c || !Cot) return;
  var v = vehiculoCot();
  var cabecera =
    '<div class="modos"><button data-modo="flete" class="' + (Cot.modo === 'flete' ? 'activo' : '') + '">' + ico('route') + 'Flete por ruta</button><button data-modo="renta" class="' + (Cot.modo === 'renta' ? 'activo' : '') + '">' + ico('calendar') + 'Renta</button></div>';
  var vehSel = '<div class="fila" style="flex-wrap:nowrap;margin-top:6px"><span class="muted">' + ico('truck') + '</span><select id="selVeh" aria-label="Vehículo">' + opcionesVehiculos(Cot.vehiculoId) + '</select></div>' +
    (v.ejemplo ? '<div class="aviso" style="margin-top:8px;font-size:12px">' + ico('alert') + '<div>Este perfil tiene <b>valores de ejemplo</b>. Ajústalo en Vehículos antes de enviar precios.</div></div>' : '');
  if (Cot.modo === 'renta') { c.innerHTML = cabecera + vehSel + htmlPanelRenta(); enlazarPanel(c); pintarResultadoRenta(); return; }

  var ultimo = Cot.paradas.length - 1;
  var paradasHtml = Cot.paradas.map(function (p, i) {
    var ok = p.lat != null;
    return '<div class="parada" draggable="true" data-pid="' + p.id + '">' +
      '<div class="linea"><span class="asa" title="Arrastra para reordenar">' + ico('grip') + '</span><span class="pin' + (ok ? ' ok' : '') + '">' + ico('pin') + '</span>' +
      '<div class="buscador"><input type="text" data-buscar="' + p.id + '" value="' + esc(p.texto) + '" placeholder="' + (i === 0 ? 'Origen: estación, TAD o lugar' : 'Destino: estación, TAD o lugar') + '" autocomplete="off"><span class="lupa">' + ico('search') + '</span></div>' +
      '<button class="btn icono fantasma" data-quitar="' + p.id + '" title="Quitar parada" aria-label="Quitar parada">' + ico('x') + '</button>' +
      '<button class="btn icono fantasma" data-menu-parada="' + p.id + '" title="Más opciones" aria-label="Más opciones">' + ico('dots') + '</button></div>' +
      (i < ultimo ? '<div class="meta"><label><input type="radio" name="carga-' + p.id + '" data-carga="' + p.id + '" value="1"' + (p.conCarga ? ' checked' : '') + '>Con carga</label><label><input type="radio" name="carga-' + p.id + '" data-carga="' + p.id + '" value="0"' + (!p.conCarga ? ' checked' : '') + '>Sin carga</label>' + (ok ? '<span class="faint">· ' + p.lat.toFixed(4) + ', ' + p.lon.toFixed(4) + '</span>' : '') + '</div>' : '') +
      '<div class="tiempos" data-tiempos="' + p.id + '">Parada programada de ' + fmt.hm(num(p.minParada) / 60) + ' y ralentí ' + fmt.hm(num(p.minRalenti) / 60) + '</div>' +
      (p.editandoTiempos ? '<div class="editor-tiempos"><div class="campo"><label>Parada (min)</label><input type="number" min="0" data-tparada="' + p.id + '" value="' + num(p.minParada) + '"></div><div class="campo"><label>Ralentí (min)</label><input type="number" min="0" data-tralenti="' + p.id + '" value="' + num(p.minRalenti) + '"></div></div>' : '') +
      '</div>';
  }).join('');

  var avisos = [];
  var pv = paradasValidas();
  if (pv.length >= 2 && distKm([pv[0].lat, pv[0].lon], [pv[pv.length - 1].lat, pv[pv.length - 1].lon]) > 1) avisos.push({ tipo: '', icono: 'undo', txt: 'Esta ruta no incluye el regreso. Si no tienes carga asegurada para la vuelta, usa <b>Trazar regreso</b> para agregar el punto de partida como destino final.' });
  if (pv.length >= 2 && !Cot.paradas.some(function (p) { return num(p.minParada) > 0; })) avisos.push({ tipo: '', icono: 'clock', txt: 'Esta ruta no tiene tiempos de parada. Considera registrar tiempos de espera por carga y descarga.' });
  if (Cot.ruta) Cot.ruta.avisos.forEach(function (a) { avisos.push({ tipo: a.tipo, icono: a.tipo === 'error' ? 'alert' : 'info', txt: esc(a.txt) }); });
  if (Directorio.error) avisos.push({ tipo: 'info', icono: 'info', txt: esc(Directorio.error) });
  var cli = clientePorId(Cot.clienteId);

  c.innerHTML = cabecera +
    '<div class="fila" style="gap:6px"><button class="btn chico" id="misRutas">' + ico('bookmark') + 'Mis rutas</button><button class="btn chico" id="limpiarTodo">' + ico('eraser') + 'Limpiar todo</button><div style="flex:1"></div><button class="btn chico suave" id="guardarRuta"' + (Cot.ruta ? '' : ' disabled') + '>' + ico('save') + 'Guardar ruta</button></div>' +
    vehSel +
    '<div class="fila" style="flex-wrap:nowrap;margin-top:8px"><span class="muted">' + ico('users') + '</span><select id="selCliente" aria-label="Cliente"><option value="">Cliente (opcional, para reglas comerciales)</option>' + D.clientes.map(function (x) { return '<option value="' + x.id + '"' + (x.id === Cot.clienteId ? ' selected' : '') + '>' + esc(x.nombre) + (x.tipo === 'grupo' ? ' · grupo' : '') + '</option>'; }).join('') + '</select></div>' +
    (cli && cli.tipo === 'grupo' && D.parametros.reglaGrupo.activa ? '<div class="aviso info" style="margin-top:8px;font-size:12px">' + ico('info') + '<div>Cliente del grupo: hasta ' + fmt.num(D.parametros.reglaGrupo.limiteKm, 0) + ' km se cobra ' + fmt.mon(D.parametros.reglaGrupo.tarifaM3) + ' por m³; arriba de ese límite, utilidad fija de ' + fmt.num(D.parametros.reglaGrupo.utilidadRutaLarga, 0) + '%.</div></div>' : '') +
    '<div class="subtitulo">Configuración de ruta</div>' +
    '<div class="grid-2"><div class="campo"><label>Tráfico</label><select id="selTrafico"><option value="con"' + (Cot.trafico === 'con' ? ' selected' : '') + '>Con tráfico</option><option value="libre"' + (Cot.trafico === 'libre' ? ' selected' : '') + '>Flujo libre</option></select></div>' +
    '<div class="campo"><label title="Límite del vehículo; 0 = sin límite">Vel. máx. (KPH) ' + ico('info') + '</label><input type="number" min="0" id="inpVmax" value="' + esc(Cot.vmax) + '"></div></div>' +
    '<div class="paradas" style="margin-top:12px" id="listaParadas">' + paradasHtml + '</div>' +
    '<div class="fila entre" style="margin-top:6px"><button class="btn fantasma" id="agregarDestino">' + ico('plus') + 'Agregar destino</button><button class="btn fantasma" id="trazarRegreso" style="color:var(--warn)">' + ico('undo') + 'Trazar regreso</button></div>' +
    avisos.map(function (a) { return '<div class="aviso ' + a.tipo + '" style="margin-top:8px;font-size:12.5px">' + ico(a.icono) + '<div>' + a.txt + '</div></div>'; }).join('') +
    '<div class="resultados-ruta" id="resRuta"></div>';
  enlazarPanel(c);
  pintarResultadosRuta();
}

function pintarResultadosRuta() {
  var c = $('#resRuta'); if (!c) return;
  var r = Cot.res;
  var km = Cot.ruta ? Cot.ruta.tramos.reduce(function (a, t) { return a + t.km; }, 0) : 0;
  var h = r ? r.tiempo.servicioH : 0;
  var peajeAuto = Cot.ruta ? (Cot.ruta.casetas || []).length : 0;
  c.innerHTML = '<div class="cab">' + ico('map') + 'Resultados de ruta' + (Cot.calculando ? ' <span class="faint" style="font-weight:400;margin-left:auto">Calculando…</span>' : '') + '</div>' +
    '<div class="f"><span title="Distancia por carretera de todos los tramos">Distancia</span><b>' + (Cot.ruta ? fmt.km(km) : '—') + '</b></div>' +
    '<div class="f"><span title="Manejo con factor de tráfico, paradas, descansos y recargas">Tiempo</span><b>' + (Cot.ruta ? fmt.dur(h) : '—') + '</b></div>' +
    '<div class="f"><span title="' + (peajeAuto ? peajeAuto + ' caseta(s) detectada(s) en tu catálogo' : 'Captura el total de casetas o registra tu catálogo de casetas en Configuración') + '">Peajes</span><div class="con-prefijo" style="width:130px"><span>$</span><input type="number" min="0" step="any" id="inpPeajes" value="' + esc(Cot.peajesManual != null && Cot.peajesManual !== '' ? Cot.peajesManual : Motor.redondear(peajesRuta(), 2)) + '" aria-label="Peajes"></div></div>' +
    '<div class="f total" id="abrirDetalle" title="Ver el detalle del costo"><span style="text-decoration:none;color:var(--text);font-weight:600">Costo total</span><b class="fila" style="gap:6px">' + (r ? fmt.mon(r.totalConIva) : '') + ico('sigma') + ico('chevR') + '</b></div>' +
    (Cot.ruta && Cot.ruta.motor ? '<div class="f" style="font-size:11.5px"><span class="faint" style="text-decoration:none">Ruteo: ' + esc(Cot.ruta.motor) + '</span></div>' : '');
  var ip = $('#inpPeajes', c);
  ip.addEventListener('input', function () { Cot.peajesManual = ip.value === '' ? null : num(ip.value); recalcularSinPintarRes(); });
  $('#abrirDetalle', c).addEventListener('click', function () { if (!Cot.ruta) { toast('Primero traza una ruta con al menos dos puntos.', 'warn'); return; } abrirDetalle('detallado'); });
}
function recalcularSinPintarRes() {
  Cot.res = Cot.ruta ? Motor.costearViaje(D, vehiculoCot(), entradaFlete()) : null;
  var b = $('#abrirDetalle b'); if (b && Cot.res) b.innerHTML = fmt.mon(Cot.res.totalConIva) + ico('sigma') + ico('chevR');
  if (Cot.detalleAbierto) actualizarSalidasDetalle();
}

function enlazarPanel(c) {
  $$('[data-modo]', c).forEach(function (b) { b.addEventListener('click', function () { Cot.modo = b.dataset.modo; cerrarDetalle(); pintarPanelViaje(); recalcular(); }); });
  var sv = $('#selVeh', c);
  if (sv) sv.addEventListener('change', function () { Cot.vehiculoId = sv.value; Cot.ajustes = ajustesDeVehiculo(vehiculoCot()); detectarCasetas(); pintarPanelViaje(); recalcular(); if (Cot.detalleAbierto) pintarDetalle(); });
  if (Cot.modo === 'renta') return;
  $('#selCliente', c).addEventListener('change', function (e) { Cot.clienteId = e.target.value; pintarPanelViaje(); recalcular(); if (Cot.detalleAbierto) pintarDetalle(); });
  $('#selTrafico', c).addEventListener('change', function (e) { Cot.trafico = e.target.value; ajustarTiempos(); recalcular(); });
  $('#inpVmax', c).addEventListener('change', function (e) { Cot.vmax = num(e.target.value); ajustarTiempos(); recalcular(); });
  $('#agregarDestino', c).addEventListener('click', function () { Cot.paradas.push(nuevaParada(true)); pintarPanelViaje(); var ins = $$('[data-buscar]'); ins[ins.length - 1].focus(); });
  $('#trazarRegreso', c).addEventListener('click', function () {
    var pv = paradasValidas(); if (!pv.length) { toast('Primero define el punto de partida.', 'warn'); return; }
    var o = pv[0]; var vac = Cot.paradas.filter(function (p) { return p.lat == null; }); vac.forEach(function (p) { Cot.paradas.splice(Cot.paradas.indexOf(p), 1); });
    var penultima = Cot.paradas[Cot.paradas.length - 1]; if (penultima) penultima.conCarga = false;
    Cot.paradas.push({ id: uid('p'), texto: o.texto + ' (regreso)', lat: o.lat, lon: o.lon, conCarga: false, minParada: 0, minRalenti: 0, origen: o.origen });
    pintarPanelViaje(); calcularRutaPronto();
  });
  $('#limpiarTodo', c).addEventListener('click', function () { var vid = Cot.vehiculoId; Cot = nuevoCot(vid); cerrarDetalle(); pintarPanelViaje(); pintarMapa(); recalcular(); });
  $('#guardarRuta', c).addEventListener('click', guardarRutaActual);
  $('#misRutas', c).addEventListener('click', abrirMisRutas);
  $$('[data-quitar]', c).forEach(function (b) { b.addEventListener('click', function () {
    var i = Cot.paradas.findIndex(function (p) { return p.id === b.dataset.quitar; });
    if (Cot.paradas.length <= 2) { Cot.paradas[i] = nuevaParada(Cot.paradas[i].conCarga); } else Cot.paradas.splice(i, 1);
    pintarPanelViaje(); calcularRutaPronto();
  }); });
  $$('[data-carga]', c).forEach(function (r) { r.addEventListener('change', function () {
    var p = Cot.paradas.find(function (x) { return x.id === r.dataset.carga; }); p.conCarga = r.value === '1';
    if (Cot.ruta) { var pv = paradasValidas(); var k = pv.indexOf(p); if (k >= 0 && Cot.ruta.tramos[k]) Cot.ruta.tramos[k].conCarga = p.conCarga; }
    pintarMapa(); recalcular();
  }); });
  $$('[data-tiempos]', c).forEach(function (el) { el.addEventListener('click', function () { var p = Cot.paradas.find(function (x) { return x.id === el.dataset.tiempos; }); p.editandoTiempos = !p.editandoTiempos; pintarPanelViaje(); }); });
  $$('[data-tparada],[data-tralenti]', c).forEach(function (el) { el.addEventListener('input', function () {
    var id = el.dataset.tparada || el.dataset.tralenti; var p = Cot.paradas.find(function (x) { return x.id === id; });
    if (el.dataset.tparada) p.minParada = Math.max(0, num(el.value)); else p.minRalenti = Math.max(0, num(el.value));
    el.closest('.parada').querySelector('.tiempos').textContent = 'Parada programada de ' + fmt.hm(p.minParada / 60) + ' y ralentí ' + fmt.hm(p.minRalenti / 60);
    recalcular();
  }); });
  $$('[data-menu-parada]', c).forEach(function (b) { b.addEventListener('click', function (e) {
    e.stopPropagation(); $$('.menu-pop').forEach(function (m) { m.remove(); });
    var id = b.dataset.menuParada, i = Cot.paradas.findIndex(function (p) { return p.id === id; });
    var pop = document.createElement('div'); pop.className = 'menu-pop'; pop.style.right = '0';
    pop.innerHTML = '<button data-a="mapa">' + ico('pin') + 'Elegir en el mapa</button>' + (i > 0 ? '<button data-a="subir">' + ico('chevU') + 'Mover arriba</button>' : '') + (i < Cot.paradas.length - 1 ? '<button data-a="bajar">' + ico('chevD') + 'Mover abajo</button>' : '') + '<button data-a="insertar">' + ico('plus') + 'Insertar parada después</button>';
    b.parentNode.style.position = 'relative'; b.parentNode.appendChild(pop);
    pop.addEventListener('click', function (ev) {
      var a = ev.target.closest('[data-a]'); if (!a) return; pop.remove();
      if (a.dataset.a === 'mapa') { Cot.eligiendo = id; $('#vistaCotizar').classList.add('cursor-mapa'); toast('Haz clic en el mapa para ubicar la parada.'); return; }
      if (a.dataset.a === 'subir') Cot.paradas.splice(i - 1, 0, Cot.paradas.splice(i, 1)[0]);
      if (a.dataset.a === 'bajar') Cot.paradas.splice(i + 1, 0, Cot.paradas.splice(i, 1)[0]);
      if (a.dataset.a === 'insertar') Cot.paradas.splice(i + 1, 0, nuevaParada(true));
      pintarPanelViaje(); calcularRutaPronto();
    });
    setTimeout(function () { document.addEventListener('click', function cerrar() { pop.remove(); document.removeEventListener('click', cerrar); }); }, 0);
  }); });
  // Arrastrar para reordenar
  var arrastrado = null;
  $$('.parada', c).forEach(function (el) {
    el.addEventListener('dragstart', function (e) { if (e.target.closest('input')) { e.preventDefault(); return; } arrastrado = el.dataset.pid; el.classList.add('arrastrando'); });
    el.addEventListener('dragend', function () { el.classList.remove('arrastrando'); });
    el.addEventListener('dragover', function (e) { e.preventDefault(); });
    el.addEventListener('drop', function (e) {
      e.preventDefault(); if (!arrastrado || arrastrado === el.dataset.pid) return;
      var de = Cot.paradas.findIndex(function (p) { return p.id === arrastrado; }), a = Cot.paradas.findIndex(function (p) { return p.id === el.dataset.pid; });
      Cot.paradas.splice(a, 0, Cot.paradas.splice(de, 1)[0]); arrastrado = null; pintarPanelViaje(); calcularRutaPronto();
    });
  });
  $$('[data-buscar]', c).forEach(enlazarBuscador);
}

/* ---------- Buscador de paradas ---------- */
function enlazarBuscador(inp) {
  var cont = inp.parentNode, lista = null, opciones = [], foco = -1;
  function cerrar() { if (lista) { lista.remove(); lista = null; } foco = -1; }
  function elegir(o) {
    var p = Cot.paradas.find(function (x) { return x.id === inp.dataset.buscar; });
    cerrar();
    if (o.accion === 'mapa') { Cot.eligiendo = p.id; $('#vistaCotizar').classList.add('cursor-mapa'); toast('Haz clic en el mapa para ubicar la parada.'); return; }
    if (o.accion === 'osm') { buscarOSM(inp.value.trim(), p); return; }
    p.texto = o.texto; p.lat = o.lat; p.lon = o.lon; p.origen = o.tipo;
    pintarPanelViaje(); calcularRutaPronto();
  }
  function mostrar() {
    var q = inp.value.trim().toLowerCase();
    opciones = [];
    var m = q.match(/^\s*(-?\d{1,2}(?:\.\d+)?)\s*[, ]\s*(-?\d{1,3}(?:\.\d+)?)\s*$/);
    if (m) opciones.push({ tipo: 'coord', texto: 'Coordenadas ' + m[1] + ', ' + m[2], lat: parseFloat(m[1]), lon: parseFloat(m[2]) });
    if (q.length >= 2) {
      Directorio.estaciones.filter(function (e) { return e.nombre.toLowerCase().indexOf(q) >= 0; }).slice(0, 6).forEach(function (e) { opciones.push({ tipo: 'estacion', texto: e.nombre, lat: e.lat, lon: e.lon }); });
      Directorio.terminales.filter(function (e) { return e.nombre.toLowerCase().indexOf(q) >= 0; }).slice(0, 6).forEach(function (e) { opciones.push({ tipo: 'tad', texto: e.nombre, lat: e.lat, lon: e.lon }); });
      D.rutas.forEach(function (r) { r.paradas.forEach(function (p) { if (p.texto && p.texto.toLowerCase().indexOf(q) >= 0 && opciones.length < 16 && !opciones.some(function (o) { return o.texto === p.texto; })) opciones.push({ tipo: 'guardada', texto: p.texto, lat: p.lat, lon: p.lon }); }); });
      opciones.push({ accion: 'osm', tipo: 'osm', texto: 'Buscar «' + inp.value.trim() + '» en el mapa (OpenStreetMap)' });
    } else if (!Directorio.cargado) {
      opciones.push({ accion: 'info', tipo: 'osm', texto: 'Escribe al menos 2 letras. Directorio de estaciones/TAD no cargado.' });
    }
    opciones.push({ accion: 'mapa', tipo: 'osm', texto: 'Elegir el punto con un clic en el mapa' });
    cerrar();
    lista = document.createElement('div'); lista.className = 'sugerencias';
    var etiquetas = { estacion: 'EST', tad: 'TAD', coord: 'XY', guardada: 'RUTA', osm: '' };
    lista.innerHTML = opciones.map(function (o, i) { return '<button type="button" data-i="' + i + '">' + (etiquetas[o.tipo] ? '<span class="tipo ' + o.tipo + '">' + etiquetas[o.tipo] + '</span>' : '<span class="tipo osm">' + ico(o.accion === 'mapa' ? 'pin' : 'search') + '</span>') + '<span>' + esc(o.texto) + (o.lat != null ? '<span class="sub">' + o.lat.toFixed(4) + ', ' + o.lon.toFixed(4) + '</span>' : '') + '</span></button>'; }).join('');
    cont.appendChild(lista);
    lista.addEventListener('mousedown', function (e) { e.preventDefault(); var b = e.target.closest('[data-i]'); if (b && opciones[+b.dataset.i].accion !== 'info') elegir(opciones[+b.dataset.i]); });
  }
  inp.addEventListener('focus', mostrar);
  inp.addEventListener('input', mostrar);
  inp.addEventListener('blur', function () { setTimeout(cerrar, 120); });
  inp.addEventListener('keydown', function (e) {
    if (!lista) return;
    var bs = $$('button', lista);
    if (e.key === 'ArrowDown' || e.key === 'ArrowUp') { e.preventDefault(); foco = (foco + (e.key === 'ArrowDown' ? 1 : -1) + bs.length) % bs.length; bs.forEach(function (b, i) { b.classList.toggle('foco', i === foco); }); }
    if (e.key === 'Enter') { e.preventDefault(); var o = opciones[foco >= 0 ? foco : 0]; if (o && o.accion !== 'info') elegir(o); }
    if (e.key === 'Escape') cerrar();
  });
}

async function buscarOSM(q, parada) {
  if (!q) return;
  toast('Buscando «' + esc(q) + '»…');
  try {
    var r = await fetch('https://nominatim.openstreetmap.org/search?format=json&countrycodes=mx&limit=6&accept-language=es&q=' + encodeURIComponent(q));
    var j = await r.json();
    if (!j.length) { toast('Sin resultados en OpenStreetMap para «' + esc(q) + '».', 'warn'); return; }
    if (j.length === 1) { aplicar(j[0]); return; }
    var m = modal({ titulo: 'Resultados para «' + q + '»', icono: 'search', cuerpo: j.map(function (x, i) { return '<button class="btn bloque" style="justify-content:flex-start;text-align:left;margin-bottom:6px;white-space:normal" data-i="' + i + '">' + ico('pin') + esc(x.display_name) + '</button>'; }).join('') });
    $$('[data-i]', m.el).forEach(function (b) { b.addEventListener('click', function () { aplicar(j[+b.dataset.i]); m.cerrar(); }); });
  } catch (e) { toast('No se pudo consultar OpenStreetMap: ' + esc(e.message), 'error'); }
  function aplicar(x) { parada.lat = parseFloat(x.lat); parada.lon = parseFloat(x.lon); parada.texto = x.display_name.split(',').slice(0, 2).join(','); parada.origen = 'osm'; pintarPanelViaje(); calcularRutaPronto(); }
}

/* ---------- Mis rutas ---------- */
async function guardarRutaActual() {
  if (!Cot.ruta) return;
  var pv = paradasValidas();
  var nombre = await pedirTexto('Guardar ruta', 'Nombre de la ruta', (pv[0] ? pv[0].texto : '') + ' → ' + (pv[pv.length - 1] ? pv[pv.length - 1].texto : ''));
  if (!nombre) return;
  var r = {
    id: uid('ruta'), nombre: nombre, creado: new Date().toISOString(), trafico: Cot.trafico, vmax: Cot.vmax,
    paradas: pv.map(function (p) { return { texto: p.texto, lat: p.lat, lon: p.lon, conCarga: p.conCarga, minParada: p.minParada, minRalenti: p.minRalenti, origen: p.origen }; }),
    tramos: Cot.ruta.tramos.map(function (t) { return { desde: t.desde, hasta: t.hasta, km: t.km, horasBase: t.horasBase, conCarga: t.conCarga, coords: simplificar(t.coords, 250), motor: t.motor, alterna: t.alterna }; }),
    avisos: Cot.ruta.avisos, peajesManual: Cot.peajesManual
  };
  D.rutas.push(r); guardar(); toast('Ruta <b>' + esc(nombre) + '</b> guardada. Úsala en Mis rutas o en la Lista de precios.', 'ok');
}
function cargarRutaGuardada(r) {
  Cot.modo = 'flete';
  Cot.paradas = r.paradas.map(function (p) { return Object.assign(nuevaParada(p.conCarga), clonar(p)); });
  Cot.trafico = r.trafico || 'con'; Cot.vmax = r.vmax || Cot.vmax; Cot.peajesManual = r.peajesManual;
  Cot.ruta = { tramos: clonar(r.tramos), avisos: clonar(r.avisos || []), motor: r.tramos[0] ? r.tramos[0].motor + ' · ruta guardada' : '' };
  ajustarTiempos(); detectarCasetas(); pintarPanelViaje(); pintarMapa(); recalcular();
}
function abrirMisRutas() {
  var m = modal({ titulo: 'Mis rutas', icono: 'bookmark', ancho: 'medio', cuerpo: '<div id="listaRutas"></div>' });
  function pintar() {
    var c = $('#listaRutas', m.el);
    if (!D.rutas.length) { c.innerHTML = '<div class="vacio-estado"><div class="circulo">' + ico('bookmark') + '</div><b>Sin rutas guardadas</b>Traza una ruta y pulsa «Guardar ruta».</div>'; return; }
    c.innerHTML = '<div class="tabla-wrap"><table class="tabla"><thead><tr><th class="sin-orden">Ruta</th><th class="sin-orden num">Km</th><th class="sin-orden">Paradas</th><th class="sin-orden">Creada</th><th class="sin-orden"></th></tr></thead><tbody>' +
      D.rutas.map(function (r) { var km = r.tramos.reduce(function (a, t) { return a + t.km; }, 0); return '<tr><td>' + esc(r.nombre) + '</td><td class="num">' + fmt.num(km, 1) + '</td><td>' + r.paradas.length + '</td><td>' + fmt.fecha(r.creado) + '</td><td class="nowrap"><button class="btn chico primario" data-usar="' + r.id + '">Usar</button> <button class="btn icono fantasma" data-ren="' + r.id + '" title="Renombrar">' + ico('edit') + '</button><button class="btn icono fantasma peligro" data-del="' + r.id + '" title="Eliminar">' + ico('trash') + '</button></td></tr>'; }).join('') + '</tbody></table></div>';
    $$('[data-usar]', c).forEach(function (b) { b.addEventListener('click', function () { cargarRutaGuardada(D.rutas.find(function (r) { return r.id === b.dataset.usar; })); m.cerrar(); }); });
    $$('[data-ren]', c).forEach(function (b) { b.addEventListener('click', async function () { var r = D.rutas.find(function (x) { return x.id === b.dataset.ren; }); var n = await pedirTexto('Renombrar ruta', 'Nombre', r.nombre); if (n) { r.nombre = n; guardar(); pintar(); } }); });
    $$('[data-del]', c).forEach(function (b) { b.addEventListener('click', async function () { if (await confirmar('¿Eliminar la ruta guardada?', 'Eliminar', true)) { D.rutas = D.rutas.filter(function (x) { return x.id !== b.dataset.del; }); guardar(); pintar(); } }); });
  }
  pintar();
}

/* ============================================================
   DETALLE DEL VIAJE (Detallado / Resumido / Cotización)
   ============================================================ */
function abrirDetalle(tab) { Cot.detalleAbierto = true; Cot.detalleTab = tab || 'detallado'; pintarDetalle(); }
function cerrarDetalle() { Cot.detalleAbierto = false; var p = $('#panelDetalle'); if (p) { p.classList.add('oculto'); p.innerHTML = ''; } }

function pintarDetalle() {
  var p = $('#panelDetalle'); if (!p) return;
  if (!Cot.res) { recalcular(); }
  if (!Cot.res) { cerrarDetalle(); return; }
  var v = vehiculoCot(), r = Cot.res;
  var scroll = (p.querySelector('.pd-cuerpo') || {}).scrollTop || 0;
  p.classList.remove('oculto');
  p.innerHTML = '<div class="pd-cab"><span>' + ico('grip') + '</span><h3>Detalle del viaje</h3>' +
    '<button class="tab' + (Cot.detalleTab === 'detallado' ? ' activa' : '') + '" data-tab="detallado">' + ico('table') + 'Detallado</button>' +
    '<button class="tab' + (Cot.detalleTab === 'resumido' ? ' activa' : '') + '" data-tab="resumido">' + ico('clock') + 'Resumido</button>' +
    '<button class="tab verde" data-tab="cotizacion">' + ico('fileText') + 'Cotización</button>' +
    '<button class="cerrar" aria-label="Cerrar detalle">' + ico('x') + '</button></div>' +
    '<div class="pd-sub"><span>' + ico('truck') + ' Veh: <b>' + esc(v.nombre) + '</b></span><span>' + ico('route') + ' Dist: <b data-out="kmTotal"></b></span><span>' + ico('clock') + ' T: <b data-out="servicio"></b></span>' + (v.capacidadLitros ? '<span>' + ico('fuel') + ' Capacidad: <b>' + fmt.ent(v.capacidadLitros) + ' L</b></span>' : '') + '</div>' +
    '<div class="pd-cuerpo">' + (Cot.detalleTab === 'resumido' ? htmlResumido() : htmlDetallado(v, r)) + '</div>';
  p.querySelector('.pd-cuerpo').scrollTop = scroll;
  $$('[data-tab]', p).forEach(function (b) { b.addEventListener('click', function () { if (b.dataset.tab === 'cotizacion') { agregarACotizacion('flete'); return; } Cot.detalleTab = b.dataset.tab; pintarDetalle(); }); });
  p.querySelector('.cerrar').addEventListener('click', cerrarDetalle);
  if (!p._enlazado) { enlazarDetalle(p); p._enlazado = true; }
  actualizarSalidasDetalle();
}

function filaR(etq, out, extra) { return '<div class="r' + (extra || '') + '"><span>' + etq + '</span><b data-out="' + out + '"></b></div>'; }
function inputCot(ruta, valor, opt) {
  opt = opt || {};
  var inp = '<input type="number" step="any" min="0" data-cot="' + ruta + '" value="' + esc(valor == null ? '' : valor) + '"' + (opt.ph != null ? ' placeholder="' + esc(opt.ph) + '"' : '') + ' aria-label="' + esc(opt.aria || ruta) + '">';
  if (opt.pre) return '<span class="con-prefijo"><span>' + opt.pre + '</span>' + inp + '</span>';
  if (opt.suf) return '<span class="con-sufijo">' + inp + '<span>' + opt.suf + '</span></span>';
  return inp;
}

function htmlDetallado(v, r) {
  var t = r.tasas, aj = Cot.ajustes, pg = aj.pago;
  var neumSub = function (l) { return fmt.ent(l.cantidad) + ' u · ' + fmt.mon(l.precio) + ' · VU ' + fmt.ent(l.vidaKm) + ' km'; };
  var col1 =
    '<div class="bloque"><div class="t">' + ico('clock') + 'Tiempo del servicio</div>' +
    filaR('Manejando', 'manejo') + filaR('Paradas programadas', 'paradas') + filaR('Descanso obligatorio', 'descansoT') + filaR('Llenando combustible', 'recarga') + filaR('Total servicio', 'servicio', ' tot') + '</div>' +
    '<div class="bloque"><div class="t" style="color:var(--accent)">' + ico('fuel') + 'Combustible</div>' +
    '<div class="r"><span>Rendimiento cargado (km/L)</span>' + inputCot('ajustes.kmLCargado', aj.kmLCargado, { ph: v.combustible.kmLCargado, aria: 'Rendimiento cargado' }) + '</div>' +
    '<div class="r"><span>Rendimiento vacío (km/L)</span>' + inputCot('ajustes.kmLVacio', aj.kmLVacio, { ph: v.combustible.kmLVacio, aria: 'Rendimiento vacío' }) + '</div>' +
    '<div class="r"><span>Precio por litro</span>' + inputCot('ajustes.precioL', aj.precioL, { pre: '$', ph: v.combustible.precioL, aria: 'Precio por litro' }) + '</div>' +
    filaR('Km cargado / vacío', 'kmCV') + filaR('Litros cargado / vacío', 'litCV') + filaR('Ralentí (<span data-out="ralentiTxt"></span>)', 'litRal') + filaR('Total litros', 'litTot') + filaR('Costo ralentí', 'costoRal') + filaR('Costo combustible', 'costoComb', ' tot') + '</div>' +
    '<div class="bloque"><div class="t" style="color:var(--ok)">' + ico('tire') + 'Neumáticos</div>' +
    '<div class="r"><span>Dirección<small>' + neumSub(v.neumaticos.direccionales) + '</small></span><b>' + fmt.mon(t.neum.direccion) + '/km</b></div>' +
    '<div class="r"><span>Tracción<small>' + neumSub(v.neumaticos.motrices) + '</small></span><b>' + fmt.mon(t.neum.traccion) + '/km</b></div>' +
    '<div class="r"><span>Arrastre<small>' + neumSub(v.neumaticos.arrastre) + '</small></span><b>' + fmt.mon(t.neum.arrastre) + '/km</b></div>' +
    '<div class="r"><span>Costo total por km</span><b>' + fmt.mon(t.neum.total) + '/km</b></div>' + filaR('Costo neumáticos ruta', 'neumRuta', ' tot') + '</div>' +
    '<div class="bloque"><div class="t" style="color:var(--warn)">' + ico('tool') + 'Mantenimiento</div>' +
    '<div class="r"><span>Intervalo (km)</span><b>' + fmt.ent(t.mttoIntervalo) + ' km</b></div><div class="r"><span>Costo por servicio</span><b>' + fmt.mon(t.mttoCosto) + '</b></div>' + filaR('Costo mantenimiento ruta', 'mttoRuta', ' tot') + '</div>' +
    '<div class="bloque"><div class="t" style="color:var(--bad)">' + ico('trendDown') + 'Depreciación <span class="faint" style="text-transform:none;letter-spacing:0">· rescate ' + fmt.num(t.rescatePct, 0) + '%</span></div>' +
    t.activos.map(function (a, i) { return '<div class="r"><span>' + esc(a.nombre === 'Vehículo' ? 'Vehículo' : a.nombre) + ' · ' + fmt.mon(a.porKm) + '/km<small>' + fmt.mon(a.depreciable) + ' (−' + fmt.num(t.rescatePct, 0) + '% resc.) · VU ' + fmt.ent(a.vidaKm) + ' km</small></span><b data-out="dep' + i + '"></b></div>'; }).join('') +
    filaR('Costo depreciación ruta', 'depRuta', ' tot') + '</div>' +
    '<div class="bloque"><div class="t" style="color:var(--violet)">' + ico('bed') + 'Descanso del operador</div>' +
    '<div class="r"><span>Frecuencia</span><b>Cada ' + fmt.num(r.descansos.frecuenciaH, 0) + ' h · ' + fmt.num(r.descansos.duracionH, 0) + ' h c/u</b></div>' + filaR('Descansos en la ruta', 'nDesc') +
    '<div class="r"><span>Costo por descanso</span>' + inputCot('ajustes.costoDescanso', aj.costoDescanso, { pre: '$', ph: v.descanso.costoParada, aria: 'Costo por descanso' }) + '</div>' + filaR('Costo descanso ruta', 'descRuta', ' tot') + '</div>' +
    '<div class="bloque"><div class="t" style="color:var(--warn)">' + ico('user') + 'Pago del operador</div>' +
    '<div class="r"><span>Operadores</span><span class="stepper"><button data-op="-1" aria-label="Menos operadores">−</button><input type="number" min="1" data-cot="ajustes.operadores" value="' + esc(aj.operadores) + '" aria-label="Operadores"><button data-op="1" aria-label="Más operadores">+</button></span></div>' +
    '<div class="r"><label class="switch"><input type="checkbox" data-sw="porKm"' + (pg.porKm ? ' checked' : '') + '><span class="pista"></span>Por kilómetro</label><b data-out="pagoKm"></b></div>' +
    (pg.porKm ? '<div class="r"><span style="padding-left:20px">$/km cargado</span>' + inputCot('ajustes.pago.kmCargado', pg.kmCargado, { pre: '$' }) + '</div><div class="r"><span style="padding-left:20px">$/km vacío</span>' + inputCot('ajustes.pago.kmVacio', pg.kmVacio, { pre: '$' }) + '</div>' : '') +
    '<div class="r"><label class="switch"><input type="checkbox" data-sw="porHora"' + (pg.porHora ? ' checked' : '') + '><span class="pista"></span>Por hora</label><b data-out="pagoHora"></b></div>' +
    (pg.porHora ? '<div class="r"><span style="padding-left:20px">$/h con carga</span>' + inputCot('ajustes.pago.horaCargado', pg.horaCargado, { pre: '$' }) + '</div><div class="r"><span style="padding-left:20px">$/h sin carga</span>' + inputCot('ajustes.pago.horaVacio', pg.horaVacio, { pre: '$' }) + '</div>' : '') +
    '<div class="r"><label class="switch"><input type="checkbox" data-sw="comisionNeto"' + (pg.comisionNeto ? ' checked' : '') + '><span class="pista"></span>Comisión % s/ neto</label>' + (pg.comisionNeto ? inputCot('ajustes.pago.pctNeto', pg.pctNeto, { suf: '%' }) : '') + '</div>' +
    '<div class="r"><label class="switch"><input type="checkbox" data-sw="comisionUtilidad"' + (pg.comisionUtilidad ? ' checked' : '') + '><span class="pista"></span>Comisión % s/ utilidad</label>' + (pg.comisionUtilidad ? inputCot('ajustes.pago.pctUtilidad', pg.pctUtilidad, { suf: '%' }) : '') + '</div>' +
    filaR('Costo en la ruta', 'pagoRuta', ' tot') + ((pg.comisionNeto || pg.comisionUtilidad) ? filaR('Comisiones (salen de tu utilidad)', 'comisiones') : '') + '</div>' +
    '<div class="bloque"><div class="t" style="color:var(--accent)">' + ico('dollar') + 'Gastos fijos</div>' +
    '<div class="r"><span>Generales (prorrateados)<small>' + fmt.mon(t.gfGenHora) + ' /hr</small></span><b data-out="gfGenRuta"></b></div>' +
    '<div class="r"><span>Específicos del vehículo<small>' + fmt.mon(t.gfEspHora) + ' /hr</small></span><b data-out="gfEspRuta"></b></div>' + filaR('Total gastos fijos', 'gfRuta', ' tot') +
    (t.categoria ? '' : '<div class="pie-nota" style="color:var(--warn)">El perfil no tiene categoría: gastos fijos en cero.</div>') + '</div>' +
    '<div class="bloque"><div class="t" style="color:var(--ok)">' + ico('box') + 'Equipamiento</div>' +
    '<div class="r"><span>Prorrateado por hora<small>' + fmt.mon(t.equipHora) + ' /hr · ' + fmt.mon(t.equipDia) + ' /día</small></span><b data-out="equipRuta"></b></div>' + filaR('Total equipamiento', 'equipRuta2', ' tot') + '</div>' +
    '<div class="bloque"><div class="t" style="color:var(--violet)">' + ico('bank') + 'Costo de capital <span class="faint" style="text-transform:none;letter-spacing:0">· ' + fmt.num(t.tasaCapitalPct, 0) + '% anual</span></div>' +
    '<div class="r"><span>Capital promedio invertido<small>' + fmt.mon(t.capitalAnual) + ' /año ÷ ' + fmt.ent(t.horasAnio) + ' h</small></span><b>' + fmt.mon(t.capitalPromedio) + '</b></div>' +
    '<div class="r"><span>Tarifa por hora</span><b>' + fmt.mon(t.capitalHora) + ' /hr</b></div>' + filaR('Costo de capital ruta', 'capRuta', ' tot') + '</div>';

  var col2 =
    '<div class="bloque"><div class="t">' + ico('chart') + 'Con utilidad (cliente)</div>' +
    filaR('Km cargados', 'kmC') + filaR('Horas con cliente', 'hCli') + filaR('Combustible (cargado + ralentí)', 'cComb') + filaR('Neumáticos (cargado)', 'cNeum') + filaR('Mantenimiento (cargado)', 'cMtto') + filaR('Depreciación (cargado)', 'cDep') +
    filaR(ico('alert') + ' Descanso operador', 'cDesc', ' alerta') + filaR('Gastos fijos generales', 'cGfGen') + filaR('Gastos fijos específicos', 'cGfEsp') + filaR('Equipamiento', 'cEquip') + filaR('Costo de capital', 'cCap') +
    filaR('Subtotal', 'cSub', ' tot') + '<div class="r"><span>Peajes<small style="font-style:italic">pass-through · sin utilidad</small></span><b data-out="peajes"></b></div></div>' +
    '<div class="bloque"><div class="fila-carga"><div class="campo"><label>Tipo de carga</label><select id="selTipoCarga">' + D.tiposCarga.map(function (x) { return '<option value="' + x.id + '"' + (x.id === Cot.tipoCargaId ? ' selected' : '') + '>' + esc(x.nombre) + '</option>'; }).join('') + '</select></div>' +
    '<div class="campo"><label style="color:var(--warn)">' + ico('alert') + 'Aumento</label>' + inputCot('aumentoPct', Cot.aumentoPct, { suf: '%' }) + '</div>' +
    '<div class="campo"><label>Toneladas</label>' + inputCot('toneladas', Cot.toneladas) + '</div></div></div>' +
    '<div class="bloque"><div class="t">' + ico('sigma') + 'Total</div>' +
    filaR('Gasto total cargado', 'tCarg') + filaR('Gasto total vacío', 'tVac') + filaR('Peajes (pass-through)', 'tPeaj') +
    '<div class="r"><label class="vacio-lbl"><input type="checkbox" id="chkPagoUtil"' + (aj.pagoConUtilidad ? ' checked' : '') + '> utilidad <span class="faint">|</span> Pago al operador</label><b data-out="tPago"></b></div>' +
    filaR('Costo total del viaje', 'tCosto', ' tot') +
    '<div class="r"><span>Otros gastos</span><button class="btn chico" id="btnOtros">' + ico('table') + '<span data-out="tOtros"></span>' + ico('chevR') + '</button></div>' +
    filaR('Gasto total', 'tGasto', ' tot') + '</div>';

  var col3 =
    '<div class="bloque"><div class="t">' + ico('truck') + 'Tramos vacíos<span class="ayuda-ico" title="Por defecto los tramos sin carga se cobran a costo. Marca «Utilidad» en los rubros a los que quieras aplicar tu margen.">' + ico('help') + '</span></div>' +
    '<div class="r"><span>Km vacíos <b data-out="kmV"></b></span><span>Horas vacío <b data-out="hV"></b></span></div>' +
    '<div class="r" style="justify-content:flex-start;gap:14px;font-size:12px"><span><span class="dot" style="background:var(--accent);display:inline-block"></span> Cobrar</span><span><span class="dot" style="background:var(--warn);display:inline-block"></span> Utilidad</span></div>' +
    Motor.RUBROS_VACIO.map(function (rb) {
      var cfg = Cot.vacios[rb.id];
      return '<div class="r"><span class="vacio-lbl"><span class="marcas"><input type="checkbox" data-vac="' + rb.id + '" data-k="cobrar"' + (cfg.cobrar ? ' checked' : '') + ' aria-label="Cobrar ' + rb.nombre + '"><input type="checkbox" class="chk-util" data-vac="' + rb.id + '" data-k="utilidad"' + (cfg.utilidad ? ' checked' : '') + (cfg.cobrar ? '' : ' disabled') + ' aria-label="Utilidad ' + rb.nombre + '"></span>' + rb.nombre + ' (vacío)</span><b data-out="v_' + rb.id + '"></b></div>';
    }).join('') + filaR('Total tramos vacíos (a costo)', 'vTot', ' tot') + '</div>' +
    '<div class="bloque"><div class="t">' + ico('dollar') + 'Precio al cliente</div>' +
    '<div class="r"><span>Utilidad %</span>' + inputCot('utilidadPct', Cot.utilidadPct, { suf: '%' }) + '</div>' + filaR('Utilidad $', 'pUtil') +
    '<div class="r"><span>Financiamiento %</span>' + inputCot('financiamientoPct', Cot.financiamientoPct, { suf: '%' }) + '</div>' + filaR('Financiamiento $', 'pFin') +
    filaR(ico('box') + ' Aumento por carga (<span data-out="aumPct"></span>)', 'pAum') + filaR('Subtotal cliente', 'pSub', ' tot') +
    '<div class="r"><span>IVA %</span>' + inputCot('ivaPct', Cot.ivaPct, { suf: '%' }) + '</div>' + filaR('IVA $', 'pIva') +
    '<div class="total-iva"><span>Total con IVA</span><button id="btnTotalPdf" title="Abrir el desglose completo en PDF">' + ico('pdf') + '<span data-out="pTotal"></span></button></div>' +
    '<div class="r" style="font-size:12px"><span>Por tonelada: <b data-out="porTon"></b></span><span>Por km: <b data-out="porKm"></b></span></div>' +
    (v.capacidadLitros ? '<div class="r" style="font-size:12px"><span>Por litro (s/IVA): <b data-out="porL"></b></span><span>Por m³ (s/IVA): <b data-out="porM3"></b></span></div>' : '') +
    '<div class="pie-nota" data-out="notaRegla"></div></div>';

  return '<div class="det-grid"><div class="det-col">' + col1 + '</div><div class="det-col">' + col2 + '</div><div class="det-col">' + col3 + '</div></div>';
}

function htmlResumido() {
  return '<div class="resumen-grid"><div><div class="dona-wrap" id="dona"></div></div>' +
    '<div class="bloque"><div class="t">' + ico('clipboard') + 'Resumen al cliente</div>' +
    filaR('Costo total del viaje', 'tCosto') + filaR('Otros gastos', 'tOtros') + filaR('Utilidad', 'pUtil') + filaR('Financiamiento', 'pFin') + filaR('Aumento por carga (<span data-out="aumPct"></span>)', 'pAum') +
    filaR('Subtotal cliente', 'pSub', ' tot') + filaR('IVA (<span data-out="ivaPctTxt"></span>)', 'pIva') +
    '<div class="total-iva"><span>Total con IVA</span><button id="btnTotalPdf" title="Abrir el desglose completo en PDF">' + ico('pdf') + '<span data-out="pTotal"></span></button></div>' +
    '<div class="r" style="font-size:12px"><span>Por km (<span data-out="kmTotal"></span>): <b data-out="porKm"></b></span><span>Por tonelada: <b data-out="porTon"></b></span></div>' +
    '<div class="pie-nota" data-out="notaRegla"></div></div></div>' +
    '<div id="leyendaDona"></div>';
}

function salidasFlete(r) {
  var t = r.tasas, c = r.cliente, va = r.vacio;
  var o = {
    kmTotal: fmt.km(r.km.total), servicio: fmt.dur(r.tiempo.servicioH),
    manejo: fmt.dur(r.tiempo.manejoH), paradas: fmt.dur(r.tiempo.paradasH), descansoT: fmt.dur(r.tiempo.descansoH), recarga: fmt.dur(r.tiempo.recargaH),
    kmCV: fmt.num(r.km.cargado, 2) + ' km / ' + fmt.num(r.km.vacio, 2) + ' km',
    litCV: fmt.num(r.combustibleDet.litrosCargado, 2) + ' L / ' + fmt.num(r.combustibleDet.litrosVacio, 2) + ' L',
    ralentiTxt: fmt.num(r.tiempo.ralentiH, 1) + 'h · ' + fmt.num(t.ralentiLh, 1) + ' L/h', litRal: fmt.num(r.combustibleDet.litrosRalenti, 2) + ' L',
    litTot: fmt.num(r.combustibleDet.litrosTotal, 2) + ' L', costoRal: fmt.mon(r.combustibleDet.costoRalenti), costoComb: fmt.mon(r.combustibleDet.costoTotal),
    neumRuta: fmt.mon(c.neumaticos + va.neumaticos), mttoRuta: fmt.mon(c.mantenimiento + va.mantenimiento), depRuta: fmt.mon(c.depreciacion + va.depreciacion),
    nDesc: String(r.descansos.n), descRuta: fmt.mon(r.descansos.costo),
    pagoKm: fmt.mon(r.pago.porKm), pagoHora: fmt.mon(r.pago.porHora), pagoRuta: fmt.mon(r.pago.total), comisiones: fmt.mon(r.pago.comisionNeto + r.pago.comisionUtilidad),
    gfGenRuta: fmt.mon(c.gfGenerales + va.gfGenerales), gfEspRuta: fmt.mon(c.gfEspecificos + va.gfEspecificos), gfRuta: fmt.mon(c.gfGenerales + va.gfGenerales + c.gfEspecificos + va.gfEspecificos),
    equipRuta: fmt.mon(c.equipamiento + va.equipamiento), equipRuta2: fmt.mon(c.equipamiento + va.equipamiento), capRuta: fmt.mon(c.capital + va.capital),
    kmC: fmt.num(r.km.cargado, 3) + ' km', hCli: fmt.num(r.tiempo.horasCliente, 2) + ' hr',
    cComb: fmt.mon(c.combustible), cNeum: fmt.mon(c.neumaticos), cMtto: fmt.mon(c.mantenimiento), cDep: fmt.mon(c.depreciacion), cDesc: fmt.mon(c.descanso),
    cGfGen: fmt.mon(c.gfGenerales), cGfEsp: fmt.mon(c.gfEspecificos), cEquip: fmt.mon(c.equipamiento), cCap: fmt.mon(c.capital), cSub: fmt.mon(c.subtotal), peajes: fmt.mon(r.peajes),
    tCarg: fmt.mon(c.subtotal), tVac: fmt.mon(r.vacioCobrado), tPeaj: fmt.mon(r.peajes), tPago: fmt.mon(r.pago.total), tCosto: fmt.mon(r.costoTotalViaje), tOtros: fmt.mon(r.otrosTotal), tGasto: fmt.mon(r.gastoTotal),
    kmV: fmt.num(r.km.vacio, 1) + ' km', hV: fmt.num(r.tiempo.horasVacio, 2) + ' hr', vTot: fmt.mon(r.vacioCobrado),
    pUtil: fmt.mon(r.utilidad), pFin: fmt.mon(r.financiamiento), aumPct: fmt.num(r.aumentoPct, 0) + '%', pAum: fmt.mon(r.aumentoCarga), pSub: fmt.mon(r.subtotalCliente),
    pIva: fmt.mon(r.iva), ivaPctTxt: fmt.num(r.ivaPct, 0) + '%', pTotal: fmt.mon(r.totalConIva),
    porTon: fmt.mon(r.porTonelada), porKm: fmt.mon(r.porKm), porL: fmt.mon(r.porLitro, 4), porM3: fmt.mon(r.porM3)
  };
  t.activos.forEach(function (a, i) { o['dep' + i] = fmt.mon(a.porKm * r.km.total); });
  Motor.RUBROS_VACIO.forEach(function (rb) { o['v_' + rb.id] = fmt.mon(va[rb.id]); });
  var nota = [];
  if (r.reglaAplicada && r.reglaAplicada.tipo === 'tarifaFija') nota.push('Regla del grupo: ruta ≤ ' + fmt.num(r.reglaAplicada.limiteKm, 0) + ' km → ' + fmt.mon(r.reglaAplicada.tarifaM3) + ' × ' + fmt.num(r.reglaAplicada.m3, 0) + ' m³. ' + (r.subtotalCliente < r.gastoTotal ? '⚠ Por debajo del costo del viaje.' : 'Margen sobre costo: ' + fmt.pct(r.margenSobreCosto * 100, 1) + '.'));
  if (r.reglaAplicada && r.reglaAplicada.tipo === 'utilidadFija') nota.push('Regla del grupo: ruta > ' + fmt.num(r.reglaAplicada.limiteKm, 0) + ' km → utilidad fija de ' + fmt.num(r.reglaAplicada.utilidadPct, 0) + '%.');
  if (r.pago.comisionNeto + r.pago.comisionUtilidad > 0) nota.push('Utilidad neta después de comisiones: ' + fmt.mon(r.utilidadNeta) + '.');
  if (r.vacioNoCobrado > 0) nota.push('Tramos vacíos no cobrados (los absorbe la empresa): ' + fmt.mon(r.vacioNoCobrado) + '.');
  o.notaRegla = nota.join(' ');
  return o;
}

function actualizarSalidasDetalle() {
  var p = $('#panelDetalle'); if (!p || !Cot.res) return;
  var o = salidasFlete(Cot.res);
  $$('[data-out]', p).forEach(function (el) { if (o[el.dataset.out] != null) el.textContent = o[el.dataset.out]; });
  if (Cot.detalleTab === 'resumido') pintarDona(Cot.res);
}

function enlazarDetalle(p) {
  p.addEventListener('input', function (e) {
    var el = e.target; if (!el.dataset.cot) return;
    var ruta = el.dataset.cot, val = el.value === '' ? null : num(el.value);
    if (ruta.indexOf('ajustes.') === 0 && ruta !== 'ajustes.operadores') setPath(Cot, ruta, val);
    else setPath(Cot, ruta, val == null ? 0 : val);
    if (ruta === 'aumentoPct') { /* aumento manual */ }
    recalcularSinPintarRes();
  });
  p.addEventListener('change', function (e) {
    var el = e.target;
    if (el.dataset.sw) { Cot.ajustes.pago[el.dataset.sw] = el.checked; recalcular(); pintarDetalle(); }
    if (el.dataset.vac) { Cot.vacios[el.dataset.vac][el.dataset.k] = el.checked; if (el.dataset.k === 'cobrar' && !el.checked) Cot.vacios[el.dataset.vac].utilidad = false; recalcular(); pintarDetalle(); }
    if (el.id === 'chkPagoUtil') { Cot.ajustes.pagoConUtilidad = el.checked; recalcular(); }
    if (el.id === 'selTipoCarga') { Cot.tipoCargaId = el.value; var tc = D.tiposCarga.find(function (x) { return x.id === el.value; }); Cot.aumentoPct = tc ? num(tc.aumentoPct) : 0; recalcular(); pintarDetalle(); }
  });
  p.addEventListener('click', function (e) {
    var b = e.target.closest('[data-op]');
    if (b) { Cot.ajustes.operadores = Math.max(1, num(Cot.ajustes.operadores) + num(b.dataset.op)); recalcular(); pintarDetalle(); return; }
    if (e.target.closest('#btnOtros')) { editarOtrosGastos(); return; }
    if (e.target.closest('#btnTotalPdf')) { pdfDesgloseViaje(); }
  });
}

function editarOtrosGastos() {
  var lista = clonar(Cot.otros);
  var m = modal({
    titulo: 'Otros gastos del viaje', icono: 'table', ancho: 'medio', cuerpo: '<div class="aviso info" style="margin-bottom:10px">' + ico('info') + '<div>Maniobras, custodias, permisos o cualquier gasto puntual. Por defecto se cobran a costo, como las casetas; si quieres ganar sobre alguno, márcalo «con utilidad».</div></div><div id="listaOtros"></div><button class="btn chico" id="agregarOtro" style="margin-top:8px">' + ico('plus') + 'Agregar gasto</button>',
    pie: '<button class="btn" data-cerrar>Cancelar</button><button class="btn primario" data-ok>Aplicar</button>'
  });
  function pintar() {
    $('#listaOtros', m.el).innerHTML = lista.length ? '<div class="lista-mini">' + lista.map(function (o, i) {
      return '<div class="item" style="grid-template-columns:2fr 1fr auto auto"><div class="campo"><label>Concepto</label><input type="text" data-i="' + i + '" data-k="concepto" value="' + esc(o.concepto) + '"></div><div class="campo"><label>Monto $</label><input type="number" step="any" data-i="' + i + '" data-k="monto" value="' + esc(o.monto) + '"></div><label class="switch" style="margin-bottom:8px"><input type="checkbox" data-i="' + i + '" data-k="conUtilidad"' + (o.conUtilidad ? ' checked' : '') + '><span class="pista"></span>Con utilidad</label><button class="btn icono fantasma peligro" data-q="' + i + '">' + ico('trash') + '</button></div>';
    }).join('') + '</div>' : '<p class="muted">Sin otros gastos.</p>';
  }
  pintar();
  m.el.addEventListener('input', function (e) { var el = e.target; if (el.dataset.k) lista[+el.dataset.i][el.dataset.k] = el.type === 'checkbox' ? el.checked : el.type === 'number' ? num(el.value) : el.value; });
  m.el.addEventListener('change', function (e) { var el = e.target; if (el.type === 'checkbox' && el.dataset.k) lista[+el.dataset.i].conUtilidad = el.checked; });
  m.el.addEventListener('click', function (e) {
    if (e.target.closest('#agregarOtro')) { lista.push({ concepto: 'Maniobras', monto: 0, conUtilidad: false }); pintar(); }
    var q = e.target.closest('[data-q]'); if (q) { lista.splice(+q.dataset.q, 1); pintar(); }
    if (e.target.closest('[data-ok]')) { Cot.otros = lista.filter(function (o) { return num(o.monto) !== 0 || o.concepto; }); m.cerrar(); recalcular(); pintarDetalle(); }
  });
}

/* ---------- Dona del resumen ---------- */
var COLORES_DONA = ['#3987e5', '#d95926', '#199e70', '#c98500', '#d55181', '#008300', '#9085e9', '#e66767'];
function componentesCosto(r) {
  var c = r.cliente, va = r.vacio, cfg = r.vacioCfg;
  function vac(k) { return cfg[k] && cfg[k].cobrar ? va[k] : 0; }
  return [
    { n: 'Combustible', v: c.combustible + vac('combustible') },
    { n: 'Neumáticos', v: c.neumaticos + vac('neumaticos') },
    { n: 'Mantenimiento', v: c.mantenimiento + vac('mantenimiento') },
    { n: 'Depreciación', v: c.depreciacion + vac('depreciacion') },
    { n: 'Gastos fijos generales', v: c.gfGenerales + vac('gfGenerales') },
    { n: 'Gastos fijos específicos', v: c.gfEspecificos + vac('gfEspecificos') },
    { n: 'Equipamiento', v: c.equipamiento + vac('equipamiento') },
    { n: 'Costo de capital', v: c.capital + vac('capital') },
    { n: 'Pago al operador', v: r.pago.total },
    { n: 'Peajes', v: r.peajes },
    { n: 'Descansos', v: c.descanso },
    { n: 'Otros gastos', v: r.otrosTotal }
  ].filter(function (x) { return x.v > 0.004; });
}
function pintarDona(r) {
  var cont = $('#dona'); if (!cont) return;
  var comp = componentesCosto(r).sort(function (a, b) { return b.v - a.v; });
  var total = comp.reduce(function (a, x) { return a + x.v; }, 0);
  var seg = comp.slice(0, 7);
  var resto = comp.slice(7);
  if (resto.length) seg.push({ n: 'Otros rubros', v: resto.reduce(function (a, x) { return a + x.v; }, 0), resto: resto });
  var R = 120, ri = 82, cx = 140, cy = 140, ang = -Math.PI / 2, svg = '';
  seg.forEach(function (s, i) {
    var a = total > 0 ? s.v / total * Math.PI * 2 : 0;
    var gap = seg.length > 1 ? 0.012 : 0;
    var a0 = ang + gap / 2, a1 = ang + a - gap / 2;
    if (a1 > a0) {
      var large = a1 - a0 > Math.PI ? 1 : 0;
      var p = function (rad, an) { return (cx + rad * Math.cos(an)).toFixed(2) + ' ' + (cy + rad * Math.sin(an)).toFixed(2); };
      var d = seg.length === 1 ? 'M ' + p(R, 0) + ' A ' + R + ' ' + R + ' 0 1 1 ' + p(R, Math.PI) + ' A ' + R + ' ' + R + ' 0 1 1 ' + p(R, 0) + ' M ' + p(ri, 0) + ' A ' + ri + ' ' + ri + ' 0 1 0 ' + p(ri, Math.PI) + ' A ' + ri + ' ' + ri + ' 0 1 0 ' + p(ri, 0) + ' Z'
        : 'M ' + p(R, a0) + ' A ' + R + ' ' + R + ' 0 ' + large + ' 1 ' + p(R, a1) + ' L ' + p(ri, a1) + ' A ' + ri + ' ' + ri + ' 0 ' + large + ' 0 ' + p(ri, a0) + ' Z';
      svg += '<path d="' + d + '" fill="' + COLORES_DONA[i] + '" fill-rule="evenodd" data-i="' + i + '" style="cursor:pointer"></path>';
    }
    ang += a;
  });
  cont.innerHTML = '<svg viewBox="0 0 280 280" role="img" aria-label="Composición del costo total del viaje">' + svg + '</svg><div class="centro"><div><b>' + fmt.mon(total) + '</b><span class="muted">Costo total</span></div></div>';
  var tip = null;
  $$('path', cont).forEach(function (pth) {
    pth.addEventListener('mousemove', function (e) {
      var s = seg[+pth.dataset.i];
      if (!tip) { tip = document.createElement('div'); tip.className = 'dona-tip'; document.body.appendChild(tip); }
      tip.innerHTML = '<b>' + esc(s.n) + '</b><br>' + fmt.mon(s.v) + ' · ' + fmt.pct(s.v / total * 100, 1) + (s.resto ? '<br><span class="muted">' + s.resto.map(function (x) { return esc(x.n); }).join(', ') + '</span>' : '');
      tip.style.left = (e.clientX + 14) + 'px'; tip.style.top = (e.clientY + 10) + 'px';
    });
    pth.addEventListener('mouseleave', function () { if (tip) { tip.remove(); tip = null; } });
  });
  var colorDe = {}; seg.forEach(function (s, i) { if (s.resto) s.resto.forEach(function (x) { colorDe[x.n] = COLORES_DONA[i]; }); else colorDe[s.n] = COLORES_DONA[i]; });
  $('#leyendaDona').innerHTML = '<div class="leyenda-dona">' + comp.map(function (x) { return '<div><span class="dot" style="background:' + colorDe[x.n] + '"></span>' + esc(x.n) + '<b>' + fmt.mon(x.v) + '</b></div>'; }).join('') + '</div>';
}

/* ---------- Agregar a cotización ---------- */
function snapshotActual(tipo) {
  var v = vehiculoCot();
  if (tipo === 'renta') {
    var rr = Cot.resRenta;
    return {
      tipo: 'renta', fecha: new Date().toISOString(), vehiculoId: v.id, vehiculoNombre: v.nombre, periodoTexto: textoPeriodoRenta(),
      entrada: { modo: 'renta', vehiculoId: v.id, renta: clonar(Cot.renta), ajustes: clonar(Cot.ajustes) },
      resultado: { disponibilidad: rr.disponibilidad.total, uso: rr.uso.total, costoBase: rr.costoBase, utilidad: rr.utilidad, subtotal: rr.subtotal, iva: rr.iva, total: rr.total, horas: rr.horas, km: rr.km }
    };
  }
  var r = Cot.res, pv = paradasValidas();
  return {
    tipo: 'flete', fecha: new Date().toISOString(), vehiculoId: v.id, vehiculoNombre: v.nombre,
    rutaTexto: pv.map(function (p) { return p.texto; }).join(' → '),
    entrada: {
      modo: 'flete', vehiculoId: v.id, clienteId: Cot.clienteId, trafico: Cot.trafico, vmax: Cot.vmax, peajesManual: Cot.peajesManual,
      paradas: clonar(pv), ruta: { tramos: Cot.ruta.tramos.map(function (t) { return Object.assign({}, t, { coords: simplificar(t.coords, 200) }); }), avisos: clonar(Cot.ruta.avisos), motor: Cot.ruta.motor },
      ajustes: clonar(Cot.ajustes), vacios: clonar(Cot.vacios), otros: clonar(Cot.otros), tipoCargaId: Cot.tipoCargaId, aumentoPct: Cot.aumentoPct, toneladas: Cot.toneladas,
      utilidadPct: Cot.utilidadPct, financiamientoPct: Cot.financiamientoPct, ivaPct: Cot.ivaPct
    },
    resultado: {
      kmTotal: r.km.total, kmCargado: r.km.cargado, kmVacio: r.km.vacio, servicioH: r.tiempo.servicioH, costoTotalViaje: r.costoTotalViaje, gastoTotal: r.gastoTotal, peajes: r.peajes,
      utilidad: r.utilidad, financiamiento: r.financiamiento, aumentoCarga: r.aumentoCarga, subtotalCliente: r.subtotalCliente, iva: r.iva, totalConIva: r.totalConIva,
      porKm: r.porKm, porLitro: r.porLitro, porM3: r.porM3, componentes: componentesCosto(r)
    }
  };
}
function cargarSnapshotEnCotizador(s) {
  var e = s.entrada; if (!e) return;
  if (!vehiculoPorId(e.vehiculoId)) { toast('El vehículo de ese cálculo ya no existe; se usará el primero del catálogo.', 'warn'); e.vehiculoId = D.vehiculos[0].id; }
  Cot = nuevoCot(e.vehiculoId);
  if (e.modo === 'renta') { Cot.modo = 'renta'; Cot.renta = clonar(e.renta); Cot.ajustes = Object.assign(ajustesDeVehiculo(vehiculoCot()), clonar(e.ajustes || {})); return; }
  ['clienteId', 'trafico', 'vmax', 'peajesManual', 'tipoCargaId', 'aumentoPct', 'toneladas', 'utilidadPct', 'financiamientoPct', 'ivaPct'].forEach(function (k) { if (e[k] !== undefined) Cot[k] = e[k]; });
  Cot.paradas = e.paradas.map(function (p) { return Object.assign(nuevaParada(p.conCarga), clonar(p)); });
  Cot.ruta = clonar(e.ruta); Cot.ajustes = clonar(e.ajustes); Cot.vacios = clonar(e.vacios); Cot.otros = clonar(e.otros || []);
  ajustarTiempos(); detectarCasetas();
  toast('Cálculo cargado y recalculado con los parámetros actuales del vehículo y del catálogo.', 'ok', 5000);
}

function agregarACotizacion(tipo) {
  var precio, desc, v = vehiculoCot();
  if (tipo === 'renta') { if (!Cot.resRenta) return; precio = Cot.resRenta.subtotal; desc = 'Renta de ' + v.nombre + ' · ' + textoPeriodoRenta(); }
  else {
    if (!Cot.res) { toast('Primero traza la ruta.', 'warn'); return; }
    var pv = paradasValidas(); precio = Cot.res.subtotalCliente;
    var corto = function (t) { return String(t || '').split(',')[0].replace(/ \(regreso\)$/, ''); };
    desc = 'Servicio de ' + v.nombre + ' de ' + corto(pv[0].texto) + ' a ' + corto(pv[pv.length - 1].texto) + (pv.length > 2 ? ' (' + (pv.length - 2) + ' parada' + (pv.length > 3 ? 's' : '') + ' intermedia' + (pv.length > 3 ? 's' : '') + ')' : '');
  }
  var borradores = D.cotizaciones.filter(function (c) { return c.estado === 'borrador'; });
  var modo = Cot.cotizacionDestinoId ? 'existente' : 'nueva';
  var m = modal({
    titulo: 'Agregar a cotización', icono: 'fileText', ancho: 'medio',
    cuerpo: '<p class="muted" style="margin:0 0 12px;font-style:italic">Crea una nueva cotización con este recorrido como primer ítem, o agrégalo a una cotización en borrador.</p>' +
      '<div class="bloque" style="padding:10px;margin-bottom:12px"><div class="fila" style="flex-wrap:nowrap"><span class="muted">' + ico('truck') + '</span><input type="text" id="acDesc" value="' + esc(desc) + '"><b class="nowrap" id="acImporte"></b></div>' +
      '<div class="fila" style="margin-top:8px"><div class="stepper"><button data-c="-1">−</button><input type="number" id="acCant" value="1" min="1"><button data-c="1">+</button></div><div class="con-prefijo" style="flex:1"><span>$</span><input type="number" step="any" id="acPrecio" value="' + Motor.redondear(precio, 2) + '"></div></div></div>' +
      '<div class="modos"><button data-m="nueva">' + ico('plus') + 'Nueva cotización</button><button data-m="existente">' + ico('clipboard') + 'Cotización existente</button></div><div id="acCuerpo"></div>',
    pie: '<button class="btn" data-cerrar>Cancelar</button><button class="btn primario" id="acOk">' + ico('plus') + 'Crear cotización</button>'
  });
  function importe() { $('#acImporte', m.el).textContent = fmt.mon(num($('#acCant', m.el).value) * num($('#acPrecio', m.el).value)); }
  function pintar() {
    $$('[data-m]', m.el).forEach(function (b) { b.classList.toggle('activo', b.dataset.m === modo); });
    $('#acOk', m.el).innerHTML = modo === 'nueva' ? ico('plus') + 'Crear cotización' : ico('plus') + 'Agregar a la cotización';
    $('#acCuerpo', m.el).innerHTML = modo === 'nueva' ?
      '<div class="grid-2"><div class="campo" style="grid-column:span 2"><label>Cliente *</label><div class="fila" style="flex-wrap:nowrap"><select id="acCliente"><option value="">Selecciona un cliente</option>' + D.clientes.map(function (c) { return '<option value="' + c.id + '"' + (c.id === Cot.clienteId ? ' selected' : '') + '>' + esc(c.nombre) + '</option>'; }).join('') + '</select><button class="btn icono" id="acNuevoCli" title="Nuevo cliente">' + ico('plus') + '</button></div></div>' +
      '<div class="campo"><label>Vigencia hasta *</label><input type="date" id="acVig" value="' + sumarDias(hoyISO(), num(D.parametros.vigenciaDias) || 30) + '"></div>' +
      '<div class="campo"><label>IVA %</label><input type="number" id="acIva" value="' + esc(tipo === 'renta' ? Cot.renta.ivaPct : Cot.ivaPct) + '"></div>' +
      '<div class="campo" style="grid-column:span 2"><label>Notas</label><textarea id="acNotas" placeholder="Opcional. Aparecerá en la cotización al exportar."></textarea></div></div>' :
      (borradores.length ? '<div class="campo"><label>Cotización en borrador</label><select id="acExistente">' + borradores.map(function (c) { return '<option value="' + c.id + '"' + (c.id === Cot.cotizacionDestinoId ? ' selected' : '') + '>' + esc(c.folio) + ' · ' + esc(nombreCliente(c.clienteId)) + ' · ' + fmt.mon(totalesCot(c).subtotal) + '</option>'; }).join('') + '</select></div>' : '<p class="muted">No hay cotizaciones en borrador.</p>');
    var nc = $('#acNuevoCli', m.el); if (nc) nc.addEventListener('click', function () { formularioCliente(function (c) { Cot.clienteId = c.id; pintar(); }); });
  }
  pintar(); importe();
  m.el.addEventListener('input', importe);
  m.el.addEventListener('click', function (e) {
    var b = e.target.closest('[data-m]'); if (b) { modo = b.dataset.m; pintar(); return; }
    var c = e.target.closest('[data-c]'); if (c) { var ic = $('#acCant', m.el); ic.value = Math.max(1, num(ic.value) + num(c.dataset.c)); importe(); return; }
    if (e.target.closest('#acOk')) crear();
  });
  function crear() {
    var item = { id: uid('it'), descripcion: $('#acDesc', m.el).value.trim() || desc, cantidad: Math.max(1, num($('#acCant', m.el).value)), precioUnitario: num($('#acPrecio', m.el).value), tipo: tipo, snapshot: snapshotActual(tipo) };
    var cot;
    if (modo === 'nueva') {
      var cli = $('#acCliente', m.el).value, vig = $('#acVig', m.el).value;
      if (!cli) { $('#acCliente', m.el).classList.add('invalido'); toast('Selecciona un cliente.', 'warn'); return; }
      if (!vig) { $('#acVig', m.el).classList.add('invalido'); toast('Indica la vigencia.', 'warn'); return; }
      cot = { id: uid('cot'), folio: siguienteFolio(), fecha: hoyISO(), vigencia: vig, clienteId: cli, estado: 'borrador', ivaPct: num($('#acIva', m.el).value), notas: $('#acNotas', m.el).value, items: [item], usuarioId: D.usuarioActivoId, creado: new Date().toISOString() };
      D.cotizaciones.push(cot);
    } else {
      var sel = $('#acExistente', m.el); if (!sel) { toast('No hay borradores disponibles.', 'warn'); return; }
      cot = D.cotizaciones.find(function (c) { return c.id === sel.value; }); cot.items.push(item);
    }
    Cot.cotizacionDestinoId = null;
    guardar(); m.cerrar();
    toast('Cotización <b>' + esc(cot.folio) + '</b> guardada con una copia del cálculo. <a href="#/cotizaciones">Ver cotizaciones</a>', 'ok', 7000);
  }
}

/* ============================================================
   RENTA / DISPOSICIÓN
   ============================================================ */
var UNIDADES_RENTA = [
  { id: 'hora', t: 'Por hora', d: 'Tarifa por hora × las horas contratadas.', u: 'horas' },
  { id: 'jornada', t: 'Jornada', d: 'Jornada laboral de tu Tiempo efectivo.', u: 'jornadas' },
  { id: 'dia', t: 'Día', d: 'Un día de servicio según tu jornada.', u: 'días' },
  { id: 'semana', t: 'Semana', d: 'Renta por semana de operación.', u: 'semanas' },
  { id: 'mensual', t: 'Mensual', d: '1/12 del año de operación, no 30 días corridos.', u: 'meses' },
  { id: 'fija', t: 'Renta fija', d: 'Precio cerrado por mes; se sugiere desde el costo.', u: 'meses' }
];
function textoPeriodoRenta() { var u = UNIDADES_RENTA.find(function (x) { return x.id === Cot.renta.unidad; }); return fmt.num(Cot.renta.cantidad, 0) + ' ' + (u ? u.u : '') + (Cot.renta.unidad === 'fija' ? ' (renta fija)' : ''); }

function htmlPanelRenta() {
  var r = Cot.renta, v = vehiculoCot(), cat = Motor.categoria(D, v.categoriaId);
  return '<div class="subtitulo">Unidad de cobro</div><div class="grid-3" style="gap:6px">' +
    UNIDADES_RENTA.map(function (u) { return '<button class="btn chico' + (r.unidad === u.id ? ' suave' : '') + '" data-unidad="' + u.id + '" title="' + esc(u.d) + '" style="' + (r.unidad === u.id ? 'border-color:var(--accent)' : '') + '">' + esc(u.t) + '</button>'; }).join('') + '</div>' +
    '<div class="grid-2" style="margin-top:10px"><div class="campo"><label>Cantidad (' + esc((UNIDADES_RENTA.find(function (u) { return u.id === r.unidad; }) || {}).u || '') + ')</label><input type="number" min="1" data-renta="cantidad" value="' + esc(r.cantidad) + '"></div>' +
    '<div class="campo"><label>Operadores (turnos)</label><input type="number" min="1" data-renta="operadores" value="' + esc(r.operadores) + '"></div>' +
    '<div class="campo" style="grid-column:span 2"><label>Sueldo mensual por operador</label><div class="con-prefijo"><span>$</span><input type="number" step="any" data-renta="sueldoOperadorMensual" value="' + esc(r.sueldoOperadorMensual) + '"></div></div></div>' +
    '<div class="aviso info" style="margin-top:8px;font-size:12px">' + ico('clock') + '<div>La jornada viene de tu <a href="#/gastos" onclick="VISTAS.gastos.tab=\'analisis\'">Tiempo efectivo</a>: ' + (cat ? fmt.ent(cat.horasDia) + ' h/día · ' + fmt.ent(cat.dias) + ' días/año · ' + fmt.ent(cat.horasAnio) + ' h/año (' + esc(cat.nombre) + ')' : '<b>el perfil no tiene categoría</b>, sin ella no se puede calcular el periodo') + '.</div></div>' +
    '<div class="subtitulo">Uso estimado (cargo variable)</div>' +
    '<div class="campo"><label>Km estimados en el periodo</label><input type="number" min="0" step="any" data-renta="kmPeriodo" value="' + esc(r.kmPeriodo) + '"></div>' +
    '<div class="fila" style="margin-top:8px;gap:6px"><span class="muted" style="font-size:12.5px">Estimar con la ruta trazada:</span><input type="number" min="0" data-renta="viajes" value="' + esc(r.viajes) + '" style="width:64px"><select data-renta-sel="frecuencia" style="width:auto"><option value="dia"' + (r.frecuencia === 'dia' ? ' selected' : '') + '>viajes/día</option><option value="semana"' + (r.frecuencia === 'semana' ? ' selected' : '') + '>viajes/semana</option><option value="mes"' + (r.frecuencia === 'mes' ? ' selected' : '') + '>viajes/mes</option></select><button class="btn chico" id="estimarUso">' + ico('route') + 'Estimar</button></div>' +
    '<div class="pila" style="gap:8px;margin-top:10px">' +
    '<label class="switch"><input type="checkbox" data-renta-chk="combustibleIncluido"' + (r.combustibleIncluido ? ' checked' : '') + '><span class="pista"></span>' + (r.combustibleIncluido ? 'Combustible: incluido en el precio' : 'Combustible: cliente pone diésel') + '</label>' +
    '<label class="switch"><input type="checkbox" data-renta-chk="casetasIncluidas"' + (r.casetasIncluidas ? ' checked' : '') + '><span class="pista"></span>' + (r.casetasIncluidas ? 'Casetas: incluidas en el precio' : 'Casetas: cliente paga') + '</label></div>' +
    (r.casetasIncluidas ? '<div class="campo" style="margin-top:8px"><label>Casetas estimadas del periodo</label><div class="con-prefijo"><span>$</span><input type="number" step="any" data-renta="casetasPeriodo" value="' + esc(r.casetasPeriodo) + '"></div></div>' : '') +
    '<div class="subtitulo">Tope de km y excedente</div><div class="grid-2"><div class="campo"><label>Tope de km (0 = sin tope)</label><input type="number" min="0" data-renta="topeKm" value="' + esc(r.topeKm) + '"></div><div class="campo"><label>Recargo excedente</label><div class="con-sufijo"><input type="number" min="0" data-renta="recargoExcedentePct" value="' + esc(r.recargoExcedentePct) + '"><span>%</span></div></div></div>' +
    '<div class="subtitulo">Precio</div><div class="grid-2"><div class="campo"><label>Utilidad %</label><div class="con-sufijo"><input type="number" step="any" data-renta="utilidadPct" value="' + esc(r.utilidadPct) + '"><span>%</span></div></div><div class="campo"><label>IVA %</label><div class="con-sufijo"><input type="number" step="any" data-renta="ivaPct" value="' + esc(r.ivaPct) + '"><span>%</span></div></div>' +
    (r.unidad === 'fija' ? '<div class="campo" style="grid-column:span 2"><label>Precio cerrado por mes (vacío = sugerido)</label><div class="con-prefijo"><span>$</span><input type="number" step="any" data-renta="precioFijoMensual" value="' + esc(r.precioFijoMensual) + '" placeholder="Sugerido"></div></div>' : '') + '</div>' +
    '<div class="resultados-ruta" id="resRenta"></div>';
}

function pintarResultadoRenta() {
  var c = $('#resRenta'); if (!c) return;
  var rr = Cot.resRenta; if (!rr) { Cot.resRenta = Motor.costearRenta(D, vehiculoCot(), Object.assign({ ajustes: Cot.ajustes }, Cot.renta)); rr = Cot.resRenta; }
  var d = rr.disponibilidad, u = rr.uso, fija = Cot.renta.unidad === 'fija';
  c.innerHTML = '<div class="cab">' + ico('calendar') + 'Resultado de la renta</div>' +
    '<div class="f"><span>Periodo</span><b>' + esc(textoPeriodoRenta()) + ' · ' + fmt.ent(rr.horas) + ' h</b></div>' +
    '<div class="bloque" style="border:0;border-radius:0"><div class="t">' + ico('lock') + 'Disponibilidad (fijo)</div>' +
    '<div class="r"><span>Gastos fijos generales</span><b>' + fmt.mon(d.gfGenerales) + '</b></div><div class="r"><span>Gastos fijos específicos</span><b>' + fmt.mon(d.gfEspecificos) + '</b></div>' +
    '<div class="r"><span>Equipamiento</span><b>' + fmt.mon(d.equipamiento) + '</b></div><div class="r"><span>Depreciación por calendario</span><b>' + fmt.mon(d.depreciacion) + '</b></div>' +
    '<div class="r"><span>Costo de capital</span><b>' + fmt.mon(d.capital) + '</b></div><div class="r"><span>Sueldo de operador(es)</span><b>' + fmt.mon(d.sueldo) + '</b></div>' +
    '<div class="r tot"><span>Disponibilidad</span><b>' + fmt.mon(d.total) + '</b></div></div>' +
    '<div class="bloque" style="border:0;border-radius:0"><div class="t">' + ico('route') + 'Uso (variable) · ' + fmt.km(rr.km, 0) + '</div>' +
    '<div class="r"><span>Combustible</span><b>' + (Cot.renta.combustibleIncluido ? fmt.mon(u.combustible) : 'Cliente pone diésel') + '</b></div><div class="r"><span>Neumáticos</span><b>' + fmt.mon(u.neumaticos) + '</b></div>' +
    '<div class="r"><span>Mantenimiento</span><b>' + fmt.mon(u.mantenimiento) + '</b></div><div class="r"><span>Casetas</span><b>' + (Cot.renta.casetasIncluidas ? fmt.mon(u.casetas) : 'Cliente paga') + '</b></div>' +
    '<div class="r tot"><span>Uso</span><b>' + fmt.mon(u.total) + '</b></div></div>' +
    '<div class="f"><span>Costo base</span><b>' + fmt.mon(rr.costoBase) + '</b></div>' +
    (fija ? '<div class="f"><span>Precio sugerido por mes</span><b>' + fmt.mon(rr.precioSugeridoMensual) + '</b></div><div class="f"><span>Margen del precio cerrado</span><b style="color:' + (rr.margen < 0 ? 'var(--bad)' : 'var(--ok)') + '">' + fmt.pct((rr.margen || 0) * 100, 1) + '</b></div>' : '') +
    '<div class="f"><span>Utilidad</span><b>' + fmt.mon(rr.utilidad) + '</b></div><div class="f"><span>Subtotal</span><b>' + fmt.mon(rr.subtotal) + '</b></div>' +
    '<div class="f"><span>IVA (' + fmt.num(rr.ivaPct, 0) + '%)</span><b>' + fmt.mon(rr.iva) + '</b></div>' +
    '<div class="f" style="font-size:16px"><span style="text-decoration:none;color:var(--text);font-weight:700">Total con IVA</span><b style="color:var(--accent)">' + fmt.mon(rr.total) + '</b></div>' +
    (rr.tope.km > 0 ? '<div class="f" style="font-size:12.5px"><span style="text-decoration:none">Excedente: ' + fmt.mon(rr.tope.tarifaKm) + '/km · ' + fmt.mon(rr.tope.tarifaHora) + '/h</span><b></b></div>' : '') +
    (rr.tope.excedenteKm > 0 ? '<div class="aviso" style="margin:8px;font-size:12.5px">' + ico('alert') + '<div>El uso estimado supera el tope por ' + fmt.km(rr.tope.excedenteKm, 0) + '. El excedente se cobraría en ' + fmt.mon(rr.tope.montoExcedente) + ' (no incluido en el precio).</div></div>' : '') +
    (rr.tasas.categoria ? '' : '<div class="aviso error" style="margin:8px;font-size:12.5px">' + ico('alert') + '<div>El vehículo no tiene categoría: no se puede calcular la renta.</div></div>') +
    '<div style="padding:10px"><button class="btn primario bloque" id="rentaACot">' + ico('fileText') + 'Agregar a cotización</button></div>';
  $('#rentaACot', c).addEventListener('click', function () { agregarACotizacion('renta'); });
}

document.addEventListener('input', function (e) {
  var el = e.target; if (!el.dataset || !el.dataset.renta || !Cot) return;
  var k = el.dataset.renta; Cot.renta[k] = k === 'precioFijoMensual' && el.value === '' ? '' : num(el.value);
  recalcular();
});
document.addEventListener('change', function (e) {
  var el = e.target; if (!el.dataset || !Cot) return;
  if (el.dataset.rentaChk) { Cot.renta[el.dataset.rentaChk] = el.checked; pintarPanelViaje(); recalcular(); }
  if (el.dataset.rentaSel) { Cot.renta[el.dataset.rentaSel] = el.value; }
});
document.addEventListener('click', function (e) {
  if (!Cot || Cot.modo !== 'renta') return;
  var b = e.target.closest('[data-unidad]');
  if (b) { Cot.renta.unidad = b.dataset.unidad; pintarPanelViaje(); recalcular(); return; }
  if (e.target.closest('#estimarUso')) {
    var rutaKm = Cot.ruta ? Cot.ruta.tramos.reduce(function (a, t) { return a + t.km; }, 0) : 0;
    if (!rutaKm) { toast('Traza una ruta representativa en «Flete por ruta» y regresa a Renta.', 'warn'); return; }
    var cat = Motor.categoria(D, vehiculoCot().categoriaId); if (!cat) { toast('El vehículo no tiene categoría.', 'warn'); return; }
    var horas = Motor.horasPeriodo(cat, Cot.renta.unidad, Cot.renta.cantidad);
    var dias = horas / cat.horasDia, semanas = horas / (cat.horasAnio / 52), meses = horas / (cat.horasAnio / 12);
    var periodos = Cot.renta.frecuencia === 'dia' ? dias : Cot.renta.frecuencia === 'semana' ? semanas : meses;
    var viajes = num(Cot.renta.viajes) * periodos;
    var hViaje = Cot.res ? Cot.res.tiempo.servicioH : 0;
    if (hViaje > 0 && viajes * hViaje > horas * 1.0001) toast('Con ' + fmt.dur(hViaje) + ' por viaje caben como máximo ' + fmt.num(horas / hViaje, 1) + ' viajes en el periodo; ajusta la frecuencia.', 'warn', 7000);
    Cot.renta.kmPeriodo = Motor.redondear(rutaKm * viajes, 0);
    Cot.renta.casetasPeriodo = Motor.redondear(peajesRuta() * viajes, 2);
    pintarPanelViaje(); recalcular();
    toast('Uso estimado: ' + fmt.num(viajes, 1) + ' viajes × ' + fmt.km(rutaKm) + ' = ' + fmt.km(Cot.renta.kmPeriodo, 0) + '.', 'ok');
  }
});

/* ============================================================
   LISTA DE PRECIOS (TARIFARIO)
   ============================================================ */
var Tarifario = { rutaId: '', vehiculos: null, utilidadPct: null, financiamientoPct: null, ivaPct: null, precioL: '', kmCargado: '', kmVacio: '', filas: null, abiertoId: null };

function calcularFilaTarifario(ruta, v, cfg) {
  var r = { tramos: clonar(ruta.tramos) };
  var factor = (ruta.trafico || 'con') === 'con' ? (num(D.parametros.factorTrafico) || 1) : 1;
  r.tramos.forEach(function (t) { var h = num(t.horasBase) * factor; if (num(ruta.vmax) > 0) h = Math.max(h, t.km / num(ruta.vmax)); t.horas = h; });
  var peajes = ruta.peajesManual != null && ruta.peajesManual !== '' ? num(ruta.peajesManual) : detectarCasetas(r, v);
  var aj = ajustesDeVehiculo(v);
  if (cfg.precioL !== '' && cfg.precioL != null) aj.precioL = num(cfg.precioL);
  if (cfg.kmCargado !== '' && cfg.kmCargado != null) aj.pago.kmCargado = num(cfg.kmCargado);
  if (cfg.kmVacio !== '' && cfg.kmVacio != null) aj.pago.kmVacio = num(cfg.kmVacio);
  var res = Motor.costearViaje(D, v, {
    tramos: r.tramos.map(function (t) { return { km: t.km, horas: t.horas, conCarga: t.conCarga }; }),
    paradas: ruta.paradas.map(function (p) { return { minParada: p.minParada, minRalenti: p.minRalenti }; }),
    peajes: peajes, ajustes: aj, vacios: Motor.vaciosPorDefecto(), otrosGastos: [],
    utilidadPct: cfg.utilidadPct, financiamientoPct: cfg.financiamientoPct, ivaPct: cfg.ivaPct, aumentoPct: 0, toneladas: 0
  });
  return {
    vehiculoId: v.id, vehiculoNombre: v.nombre, capacidadLitros: num(v.capacidadLitros), ejemplo: !!v.ejemplo,
    km: res.km.total, horas: res.tiempo.servicioH, peajes: res.peajes, costoTotal: res.gastoTotal, utilidad: res.utilidad, financiamiento: res.financiamiento,
    subtotal: res.subtotalCliente, iva: res.iva, total: res.totalConIva, porKm: res.porKm, porLitro: res.porLitro, porM3: res.porM3,
    estrategia: { precioL: cfg.precioL, kmCargado: cfg.kmCargado, kmVacio: cfg.kmVacio, utilidadPct: cfg.utilidadPct, financiamientoPct: cfg.financiamientoPct, ivaPct: cfg.ivaPct }
  };
}

VISTAS.precios = function (cont) {
  var T = Tarifario;
  if (T.vehiculos == null) T.vehiculos = D.vehiculos.map(function (v) { return v.id; });
  if (T.utilidadPct == null) { T.utilidadPct = num(D.parametros.utilidadPct); T.financiamientoPct = num(D.parametros.financiamientoPct); T.ivaPct = num(D.parametros.ivaPct); }
  var abierto = T.abiertoId ? D.tarifarios.find(function (x) { return x.id === T.abiertoId; }) : null;
  cont.innerHTML =
    '<h2 style="margin:0;font-size:18px">Tarifario / Lista de precios</h2><div class="muted" style="margin-bottom:14px">Precios por ruta guardada y vehículo. Guarda para fijar los valores.</div>' +
    '<div class="tarjeta"><div class="cuerpo"><div class="fila" style="align-items:flex-end;gap:10px">' +
    '<div class="campo" style="width:230px"><label>Ruta guardada</label><select id="tRuta"><option value="">Elegir ruta…</option>' + D.rutas.map(function (r) { return '<option value="' + r.id + '"' + (r.id === T.rutaId ? ' selected' : '') + '>' + esc(r.nombre) + '</option>'; }).join('') + '</select></div>' +
    '<div class="campo" style="flex:1;min-width:260px;position:relative"><label>Vehículos (' + T.vehiculos.length + ')</label><button class="btn" id="tVeh" style="justify-content:space-between;width:100%;overflow:hidden">' + ico('truck') + '<span style="overflow:hidden;text-overflow:ellipsis;flex:1;text-align:left">' + esc(D.vehiculos.filter(function (v) { return T.vehiculos.indexOf(v.id) >= 0; }).map(function (v) { return v.nombre; }).join(', ') || 'Ninguno') + '</span>' + ico('chevD') + '</button></div>' +
    '<div class="campo" style="width:90px"><label>Utilidad %</label><input type="number" step="any" id="tUtil" value="' + T.utilidadPct + '"></div>' +
    '<div class="campo" style="width:90px"><label>Financ. %</label><input type="number" step="any" id="tFin" value="' + T.financiamientoPct + '"></div>' +
    '<div class="campo" style="width:80px"><label>IVA %</label><input type="number" step="any" id="tIva" value="' + T.ivaPct + '"></div>' +
    '</div><div class="fila" style="align-items:flex-end;gap:10px;margin-top:10px">' +
    '<div class="campo" style="width:150px"><label>Precio diésel $/L</label><input type="number" step="any" id="tDiesel" value="' + esc(T.precioL) + '" placeholder="Del perfil"></div>' +
    '<div class="campo" style="width:160px"><label>Operador $/km cargado</label><input type="number" step="any" id="tKmC" value="' + esc(T.kmCargado) + '" placeholder="Del perfil"></div>' +
    '<div class="campo" style="width:160px"><label>Operador $/km vacío</label><input type="number" step="any" id="tKmV" value="' + esc(T.kmVacio) + '" placeholder="Del perfil"></div>' +
    '<div style="flex:1"></div><button class="btn primario" id="tCalc">' + ico('clipboard') + 'Calcular tarifario</button><button class="btn" id="tGuardar"' + (T.filas ? '' : ' disabled') + '>' + ico('save') + 'Guardar</button></div>' +
    '<p class="faint" style="font-size:12px;margin:10px 0 0">' + ico('info') + ' «Aplicar a todas las filas»: estos valores se aplican a todos los vehículos, pero se respeta el rendimiento (km/L) de cada uno. Los tramos vacíos se cobran a costo y no hay otros gastos.</p>' +
    '</div></div>' +
    '<div class="tarjeta" style="margin-top:14px"><div class="cuerpo" id="tResultado"></div></div>' +
    '<h3 style="margin:22px 0 10px;font-size:15px">Listas de precios guardadas</h3><div id="tGuardadas"></div>';

  function pintarResultado(filas, info) {
    var c = $('#tResultado');
    if (!filas) { c.innerHTML = '<div class="vacio-estado">' + (D.rutas.length ? 'Elige una ruta guardada y pulsa «Calcular tarifario» para generar precios por vehículo.' : 'Aún no tienes rutas guardadas. Traza una ruta en <a href="#/cotizar">Cotizar servicio</a> y pulsa «Guardar ruta».') + '</div>'; return; }
    c.innerHTML = (info ? '<div class="aviso info" style="margin-bottom:10px">' + ico('lock') + '<div>' + info + '</div></div>' : '') +
      '<div class="tabla-wrap"><table class="tabla"><thead><tr>' + ['Vehículo', 'Km', 'Tiempo', 'Peajes', 'Costo del viaje', 'Utilidad', 'Subtotal', 'IVA', 'Total con IVA', 'Por km', 'Por litro', 'Por m³'].map(function (h, i) { return '<th class="sin-orden' + (i ? ' num' : '') + '">' + h + '</th>'; }).join('') + '</tr></thead><tbody>' +
      filas.map(function (f) { return '<tr><td>' + esc(f.vehiculoNombre) + (f.ejemplo ? ' <span class="badge ambar">ejemplo</span>' : '') + '</td><td class="num">' + fmt.num(f.km, 1) + '</td><td class="num">' + fmt.dur(f.horas) + '</td><td class="num">' + fmt.mon(f.peajes) + '</td><td class="num">' + fmt.mon(f.costoTotal) + '</td><td class="num">' + fmt.mon(f.utilidad) + '</td><td class="num">' + fmt.mon(f.subtotal) + '</td><td class="num">' + fmt.mon(f.iva) + '</td><td class="num"><b>' + fmt.mon(f.total) + '</b></td><td class="num">' + fmt.mon(f.porKm) + '</td><td class="num">' + (f.capacidadLitros ? fmt.mon(f.porLitro, 4) : '—') + '</td><td class="num">' + (f.capacidadLitros ? fmt.mon(f.porM3) : '—') + '</td></tr>'; }).join('') +
      '</tbody></table></div><p class="faint" style="font-size:12px">Por litro y por m³ se calculan sobre el subtotal (sin IVA) y la capacidad del autotanque.</p>';
  }
  function pintarGuardadas() {
    var c = $('#tGuardadas');
    if (!D.tarifarios.length) { c.innerHTML = '<div class="tabla-wrap"><div class="vacio-estado">Todavía no hay listas guardadas.</div></div>'; return; }
    c.innerHTML = '<div class="tabla-wrap"><table class="tabla"><thead><tr><th class="sin-orden">Nombre</th><th class="sin-orden">Ruta</th><th class="sin-orden">Fecha</th><th class="sin-orden num">Vehículos</th><th class="sin-orden"></th></tr></thead><tbody>' +
      D.tarifarios.slice().reverse().map(function (t) { return '<tr><td>' + esc(t.nombre) + '</td><td>' + esc(t.rutaNombre) + '</td><td>' + fmt.fechaHora(t.fecha) + '</td><td class="num">' + t.filas.length + '</td><td class="nowrap">' +
        '<button class="btn chico" data-tab-abrir="' + t.id + '">' + ico('eye') + 'Abrir</button> <button class="btn chico" data-tab-recalc="' + t.id + '" title="Recalcular a datos de hoy">' + ico('refresh') + 'Recalcular</button> ' +
        '<button class="btn icono fantasma" data-tab-xls="' + t.id + '" title="Exportar a Excel">' + ico('table') + '</button><button class="btn icono fantasma" data-tab-pdf="' + t.id + '" title="Exportar a PDF">' + ico('pdf') + '</button><button class="btn icono fantasma" data-tab-word="' + t.id + '" title="Exportar a Word">' + ico('word') + '</button><button class="btn icono fantasma peligro" data-tab-del="' + t.id + '" title="Eliminar">' + ico('trash') + '</button></td></tr>'; }).join('') +
      '</tbody></table></div>';
  }
  if (abierto) pintarResultado(abierto.filas, 'Lista guardada «' + esc(abierto.nombre) + '» del ' + fmt.fechaHora(abierto.fecha) + ' con valores fijados. Pulsa «Recalcular» para correrla con los datos de hoy.');
  else pintarResultado(T.filas);
  pintarGuardadas();

  function leerCfg() {
    T.rutaId = $('#tRuta').value; T.utilidadPct = num($('#tUtil').value); T.financiamientoPct = num($('#tFin').value); T.ivaPct = num($('#tIva').value);
    T.precioL = $('#tDiesel').value; T.kmCargado = $('#tKmC').value; T.kmVacio = $('#tKmV').value;
    return { precioL: T.precioL, kmCargado: T.kmCargado, kmVacio: T.kmVacio, utilidadPct: T.utilidadPct, financiamientoPct: T.financiamientoPct, ivaPct: T.ivaPct };
  }
  $('#tCalc').addEventListener('click', function () {
    var cfg = leerCfg(); T.abiertoId = null;
    var ruta = D.rutas.find(function (r) { return r.id === T.rutaId; });
    if (!ruta) { toast('Elige una ruta guardada.', 'warn'); return; }
    var vs = D.vehiculos.filter(function (v) { return T.vehiculos.indexOf(v.id) >= 0; });
    if (!vs.length) { toast('Elige al menos un vehículo.', 'warn'); return; }
    T.filas = vs.map(function (v) { return calcularFilaTarifario(ruta, v, cfg); });
    VISTAS.precios(cont);
  });
  $('#tGuardar').addEventListener('click', async function () {
    var ruta = D.rutas.find(function (r) { return r.id === T.rutaId; }); if (!T.filas || !ruta) return;
    var nombre = await pedirTexto('Guardar lista de precios', 'Nombre', 'Lista ' + ruta.nombre + ' · ' + fmt.fecha(new Date().toISOString()));
    if (!nombre) return;
    var t = { id: uid('tar'), nombre: nombre, fecha: new Date().toISOString(), rutaId: ruta.id, rutaNombre: ruta.nombre, rutaKm: ruta.tramos.reduce(function (a, x) { return a + x.km; }, 0), config: leerCfg(), filas: clonar(T.filas) };
    D.tarifarios.push(t); guardar(); T.abiertoId = t.id; toast('Lista guardada con valores fijados.', 'ok'); VISTAS.precios(cont);
  });
  $('#tVeh').addEventListener('click', function (e) {
    e.stopPropagation();
    var m = modal({ titulo: 'Vehículos del tarifario', icono: 'truck', cuerpo: D.vehiculos.map(function (v) { return '<label class="fila" style="padding:6px 0;cursor:pointer"><input type="checkbox" value="' + v.id + '"' + (T.vehiculos.indexOf(v.id) >= 0 ? ' checked' : '') + '>' + esc(v.nombre) + '</label>'; }).join(''), pie: '<button class="btn" id="todos">Todos</button><button class="btn" data-cerrar>Cancelar</button><button class="btn primario" data-ok>Aplicar</button>' });
    m.el.querySelector('#todos').addEventListener('click', function () { $$('input[type=checkbox]', m.el).forEach(function (x) { x.checked = true; }); });
    m.el.querySelector('[data-ok]').addEventListener('click', function () { leerCfg(); T.vehiculos = $$('input:checked', m.el).map(function (x) { return x.value; }); m.cerrar(); VISTAS.precios(cont); });
  });
  $('#tGuardadas').addEventListener('click', async function (e) {
    var b = e.target.closest('button'); if (!b) return;
    var id = b.dataset.tabAbrir || b.dataset.tabRecalc || b.dataset.tabXls || b.dataset.tabPdf || b.dataset.tabWord || b.dataset.tabDel;
    var t = D.tarifarios.find(function (x) { return x.id === id; }); if (!t) return;
    if (b.dataset.tabAbrir) { T.abiertoId = t.id; VISTAS.precios(cont); }
    if (b.dataset.tabRecalc) {
      var ruta = D.rutas.find(function (r) { return r.id === t.rutaId; });
      if (!ruta) { toast('La ruta de esa lista ya no existe.', 'error'); return; }
      var nuevas = t.filas.map(function (f) { var v = vehiculoPorId(f.vehiculoId); return v ? calcularFilaTarifario(ruta, v, f.estrategia || t.config) : null; }).filter(Boolean);
      var cambios = nuevas.map(function (f) { var vieja = t.filas.find(function (x) { return x.vehiculoId === f.vehiculoId; }); return esc(f.vehiculoNombre) + ': ' + fmt.mon(vieja ? vieja.total : 0) + ' → <b>' + fmt.mon(f.total) + '</b>'; }).join('<br>');
      if (await confirmar('Recalcular a datos de hoy (combustible, rendimiento y tarifas actuales de cada vehículo):<br><br>' + cambios + '<br><br>¿Actualizar la lista guardada?', 'Actualizar')) { t.filas = nuevas; t.fecha = new Date().toISOString(); guardar(); T.abiertoId = t.id; VISTAS.precios(cont); }
    }
    if (b.dataset.tabXls) exportarTarifarioExcel(t);
    if (b.dataset.tabPdf) exportarTarifarioPDF(t);
    if (b.dataset.tabWord) exportarTarifarioWord(t);
    if (b.dataset.tabDel && await confirmar('¿Eliminar la lista «' + esc(t.nombre) + '»?', 'Eliminar', true)) { D.tarifarios = D.tarifarios.filter(function (x) { return x !== t; }); if (T.abiertoId === t.id) T.abiertoId = null; guardar(); VISTAS.precios(cont); }
  });
};

/* ============================================================
   EXPORTACIONES: PDF, Excel y Word (plantillas .docx)
   ============================================================ */
var AZUL = [15, 76, 129];

function encabezadoPDF(doc, titulo, subtitulo) {
  var e = D.empresa, x = 14;
  if (e.logo) { try { doc.addImage(e.logo, 14, 10, 22, 22); x = 40; } catch (err) { /* logo inválido */ } }
  doc.setFont('helvetica', 'bold'); doc.setFontSize(14); doc.setTextColor(AZUL[0], AZUL[1], AZUL[2]);
  doc.text(e.nombre || 'Mi empresa', x, 17);
  doc.setFont('helvetica', 'normal'); doc.setFontSize(8.5); doc.setTextColor(90, 90, 90);
  var l = [e.legalName, e.rfc ? 'RFC: ' + e.rfc : '', e.direccion, [e.telefono ? 'Tel: ' + e.telefono : '', e.email].filter(Boolean).join(' · ')].filter(Boolean);
  l.forEach(function (t, i) { doc.text(String(t), x, 22 + i * 4); });
  var ancho = doc.internal.pageSize.getWidth();
  doc.setFont('helvetica', 'bold'); doc.setFontSize(titulo.length > 14 ? 13 : 20); doc.setTextColor(30, 30, 30);
  doc.text(titulo, ancho - 14, 17, { align: 'right' });
  if (subtitulo) { doc.setFont('helvetica', 'normal'); doc.setFontSize(9); doc.setTextColor(90, 90, 90); subtitulo.forEach(function (t, i) { doc.text(t, ancho - 14, 23 + i * 4.5, { align: 'right' }); }); }
  return 22 + Math.max(l.length * 4, (subtitulo || []).length * 4.5) + 6;
}
function piePDF(doc, texto) {
  var n = doc.internal.getNumberOfPages(), w = doc.internal.pageSize.getWidth(), h = doc.internal.pageSize.getHeight();
  for (var i = 1; i <= n; i++) {
    doc.setPage(i); doc.setFont('helvetica', 'italic'); doc.setFontSize(7.5); doc.setTextColor(130, 130, 130);
    doc.text(texto, 14, h - 8); doc.text('Página ' + i + ' de ' + n, w - 14, h - 8, { align: 'right' });
  }
}
var estiloTabla = { styles: { fontSize: 8, cellPadding: 1.6, lineColor: [215, 222, 228], lineWidth: 0.1 }, headStyles: { fillColor: AZUL, textColor: 255, fontStyle: 'bold' }, alternateRowStyles: { fillColor: [246, 248, 250] }, margin: { left: 14, right: 14 } };

/* ---------- Imagen del mapa para exportar ---------- */
function diagramaRuta(tramos, ancho, alto) {
  var cv = document.createElement('canvas'); cv.width = ancho; cv.height = alto;
  var ctx = cv.getContext('2d'); ctx.fillStyle = '#f4f6f8'; ctx.fillRect(0, 0, ancho, alto);
  var pts = []; tramos.forEach(function (t) { pts = pts.concat(t.coords || []); });
  if (pts.length < 2) return cv.toDataURL('image/png');
  var la = pts.map(function (p) { return p[0]; }), lo = pts.map(function (p) { return p[1]; });
  var mnLa = Math.min.apply(null, la), mxLa = Math.max.apply(null, la), mnLo = Math.min.apply(null, lo), mxLo = Math.max.apply(null, lo);
  var m = 50, rLa = Math.max(mxLa - mnLa, 1e-4), rLo = Math.max(mxLo - mnLo, 1e-4);
  function pr(p) { return [m + (p[1] - mnLo) / rLo * (ancho - 2 * m), alto - m - (p[0] - mnLa) / rLa * (alto - 2 * m)]; }
  tramos.forEach(function (t) {
    ctx.strokeStyle = t.conCarga ? '#0f4c81' : '#8a919b'; ctx.lineWidth = t.conCarga ? 7 : 5; ctx.setLineDash(t.conCarga ? [] : [12, 10]); ctx.lineJoin = 'round';
    ctx.beginPath(); (t.coords || []).forEach(function (p, i) { var q = pr(p); if (i) ctx.lineTo(q[0], q[1]); else ctx.moveTo(q[0], q[1]); }); ctx.stroke();
  });
  ctx.setLineDash([]);
  var o = pr(pts[0]), d = pr(pts[pts.length - 1]);
  [[o, '#16a34a'], [d, '#dc2626']].forEach(function (x) { ctx.fillStyle = x[1]; ctx.beginPath(); ctx.arc(x[0][0], x[0][1], 11, 0, Math.PI * 2); ctx.fill(); });
  ctx.fillStyle = '#64748b'; ctx.font = '16px Arial'; ctx.textAlign = 'right'; ctx.fillText('Representación esquemática de la ruta', ancho - 12, alto - 12);
  return cv.toDataURL('image/png');
}
async function imagenMapa(tramos) {
  var ancho = 1000, alto = 520;
  if (typeof leafletImage !== 'function' || typeof L === 'undefined') return diagramaRuta(tramos, ancho, alto);
  var div = document.createElement('div');
  div.style.cssText = 'position:fixed;left:-12000px;top:0;width:' + ancho + 'px;height:' + alto + 'px';
  document.body.appendChild(div);
  var mp = null;
  try {
    mp = L.map(div, { zoomControl: false, attributionControl: false, preferCanvas: true, fadeAnimation: false, zoomAnimation: false });
    var capa = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 19, crossOrigin: 'anonymous' }).addTo(mp);
    var rend = L.canvas({ padding: 0.5 }), todos = [];
    tramos.forEach(function (t) { if (t.coords && t.coords.length) { L.polyline(t.coords, { color: t.conCarga ? '#0f4c81' : '#6b7280', weight: 6, dashArray: t.conCarga ? null : '10 10', renderer: rend }).addTo(mp); todos = todos.concat(t.coords); } });
    if (todos.length < 2) return diagramaRuta(tramos, ancho, alto);
    L.circleMarker(todos[0], { radius: 9, color: '#fff', weight: 3, fillColor: '#16a34a', fillOpacity: 1, renderer: rend }).addTo(mp);
    L.circleMarker(todos[todos.length - 1], { radius: 9, color: '#fff', weight: 3, fillColor: '#dc2626', fillOpacity: 1, renderer: rend }).addTo(mp);
    mp.fitBounds(L.latLngBounds(todos), { padding: [40, 40], animate: false });
    await new Promise(function (res) { var hecho = false; function fin() { if (!hecho) { hecho = true; res(); } } capa.once('load', fin); setTimeout(fin, 4500); });
    await new Promise(function (res) { setTimeout(res, 250); });
    return await new Promise(function (res, rej) {
      leafletImage(mp, function (err, canvas) {
        if (err || !canvas) { rej(err || new Error('captura')); return; }
        var ctx = canvas.getContext('2d'); ctx.fillStyle = 'rgba(255,255,255,.88)'; ctx.fillRect(canvas.width - 220, canvas.height - 26, 214, 20);
        ctx.fillStyle = '#475569'; ctx.font = '13px Arial'; ctx.textAlign = 'right'; ctx.fillText('© OpenStreetMap contributors', canvas.width - 12, canvas.height - 11);
        try { res(canvas.toDataURL('image/png')); } catch (e2) { rej(e2); }
      });
    });
  } catch (e) { return diagramaRuta(tramos, ancho, alto); }
  finally { if (mp) mp.remove(); div.remove(); }
}

/* ---------- PDF: desglose completo del viaje ---------- */
async function pdfDesgloseViaje() {
  if (!libsOk(['jspdf'])) return;
  var r = Cot.res, v = vehiculoCot(); if (!r) return;
  toast('Generando el PDF del desglose…');
  var doc = new window.jspdf.jsPDF({ unit: 'mm', format: 'letter' });
  var pv = paradasValidas();
  var y = encabezadoPDF(doc, 'DESGLOSE DEL VIAJE', ['Fecha: ' + fmt.fecha(hoyISO()), 'Elaboró: ' + usuarioActivo().nombre]);
  var t = r.tasas, c = r.cliente, va = r.vacio;
  doc.autoTable(Object.assign({}, estiloTabla, {
    startY: y, head: [['Datos del viaje', '']], columnStyles: { 0: { cellWidth: 45, fontStyle: 'bold' } },
    body: [['Vehículo', v.nombre + (v.ejemplo ? ' (perfil con valores de ejemplo)' : '')], ['Ruta', pv.map(function (p, i) { return (i + 1) + '. ' + p.texto + (i < pv.length - 1 ? (p.conCarga ? ' [con carga]' : ' [sin carga]') : ''); }).join('\n')],
      ['Distancia', fmt.km(r.km.total, 2) + ' (cargado ' + fmt.km(r.km.cargado, 2) + ' · vacío ' + fmt.km(r.km.vacio, 2) + ')'], ['Tiempo de servicio', fmt.dur(r.tiempo.servicioH) + ' (manejo ' + fmt.dur(r.tiempo.manejoH) + ', paradas ' + fmt.dur(r.tiempo.paradasH) + ', descansos ' + fmt.dur(r.tiempo.descansoH) + ', recargas ' + fmt.dur(r.tiempo.recargaH) + ')'],
      ['Ruteo', (Cot.ruta && Cot.ruta.motor) || '—'], ['Cliente', nombreCliente(Cot.clienteId)]]
  }));
  var filasRubro = [
    ['Combustible', c.combustible, va.combustible], ['Neumáticos', c.neumaticos, va.neumaticos], ['Mantenimiento', c.mantenimiento, va.mantenimiento], ['Depreciación', c.depreciacion, va.depreciacion],
    ['Descanso del operador', c.descanso, 0], ['Gastos fijos generales', c.gfGenerales, va.gfGenerales], ['Gastos fijos específicos', c.gfEspecificos, va.gfEspecificos], ['Equipamiento', c.equipamiento, va.equipamiento], ['Costo de capital', c.capital, va.capital]
  ];
  doc.autoTable(Object.assign({}, estiloTabla, {
    startY: doc.lastAutoTable.finalY + 5, head: [['Rubro', 'Con utilidad (cliente)', 'Tramos vacíos', 'Vacío cobrado', 'Total ruta']],
    columnStyles: { 1: { halign: 'right' }, 2: { halign: 'right' }, 3: { halign: 'center' }, 4: { halign: 'right' } },
    body: filasRubro.map(function (f, i) {
      var id = Motor.RUBROS_VACIO.map(function (x) { return x.nombre; }).indexOf(f[0]);
      var cfg = id >= 0 ? r.vacioCfg[Motor.RUBROS_VACIO[id].id] : null;
      return [f[0], fmt.mon(f[1]), fmt.mon(f[2]), cfg ? (cfg.cobrar ? (cfg.utilidad ? 'Sí, con utilidad' : 'Sí, a costo') : 'No') : '—', fmt.mon(f[1] + f[2])];
    }).concat([[{ content: 'Subtotal', styles: { fontStyle: 'bold' } }, { content: fmt.mon(c.subtotal), styles: { fontStyle: 'bold' } }, fmt.mon(r.vacioCobrado + r.vacioNoCobrado), 'Cobrado: ' + fmt.mon(r.vacioCobrado), '']])
  }));
  doc.autoTable(Object.assign({}, estiloTabla, {
    startY: doc.lastAutoTable.finalY + 5, head: [['Del costo al precio', 'Importe']], columnStyles: { 1: { halign: 'right' } },
    body: [['Gasto total cargado', fmt.mon(c.subtotal)], ['Gasto total vacío (cobrado)', fmt.mon(r.vacioCobrado)], ['Peajes (pass-through, sin utilidad)', fmt.mon(r.peajes)],
      ['Pago al operador' + (r.pago.conUtilidad ? ' (con utilidad)' : ' (sin utilidad)'), fmt.mon(r.pago.total)], ['Costo total del viaje', fmt.mon(r.costoTotalViaje)]]
      .concat(r.otros.map(function (o) { return ['Otro gasto: ' + o.concepto + (o.conUtilidad ? ' (con utilidad)' : ''), fmt.mon(o.monto)]; }))
      .concat([['Gasto total', fmt.mon(r.gastoTotal)], ['Utilidad (' + fmt.num(r.utilidadPct, 1) + '% sobre ' + fmt.mon(r.baseUtilidad) + ')', fmt.mon(r.utilidad)], ['Financiamiento (' + fmt.num(r.financiamientoPct, 1) + '%)', fmt.mon(r.financiamiento)],
        ['Aumento por tipo de carga (' + fmt.num(r.aumentoPct, 1) + '%)', fmt.mon(r.aumentoCarga)], [{ content: 'Subtotal cliente', styles: { fontStyle: 'bold' } }, { content: fmt.mon(r.subtotalCliente), styles: { fontStyle: 'bold' } }],
        ['IVA (' + fmt.num(r.ivaPct, 0) + '%)', fmt.mon(r.iva)], [{ content: 'Total con IVA', styles: { fontStyle: 'bold', textColor: AZUL } }, { content: fmt.mon(r.totalConIva), styles: { fontStyle: 'bold', textColor: AZUL } }],
        ['Por km (' + fmt.km(r.km.total) + ')', fmt.mon(r.porKm)], ['Por tonelada', r.porTonelada ? fmt.mon(r.porTonelada) : '—']])
      .concat(v.capacidadLitros ? [['Por litro (sin IVA, ' + fmt.ent(v.capacidadLitros) + ' L)', fmt.mon(r.porLitro, 4)], ['Por m³ (sin IVA)', fmt.mon(r.porM3)]] : [])
      .concat(r.reglaAplicada ? [['Regla comercial del grupo', r.reglaAplicada.tipo === 'tarifaFija' ? 'Tarifa fija ' + fmt.mon(r.reglaAplicada.tarifaM3) + '/m³' : 'Utilidad fija ' + fmt.num(r.reglaAplicada.utilidadPct, 0) + '%']] : [])
      .concat(r.pago.comisionNeto + r.pago.comisionUtilidad > 0 ? [['Comisiones al operador (salen de la utilidad)', fmt.mon(r.pago.comisionNeto + r.pago.comisionUtilidad)], ['Utilidad neta', fmt.mon(r.utilidadNeta)]] : [])
  }));
  doc.autoTable(Object.assign({}, estiloTabla, {
    startY: doc.lastAutoTable.finalY + 5, head: [['Parámetros usados', 'Valor']], columnStyles: { 1: { halign: 'right' } },
    body: [['Rendimiento cargado / vacío', fmt.num(t.kmLCargado, 2) + ' / ' + fmt.num(t.kmLVacio, 2) + ' km/L'], ['Precio del combustible', fmt.mon(t.precioL) + '/L'], ['Litros totales (incl. ralentí)', fmt.num(r.combustibleDet.litrosTotal, 2) + ' L'],
      ['Neumáticos', fmt.mon(t.neum.total) + '/km'], ['Mantenimiento', fmt.mon(t.mttoKm) + '/km'], ['Depreciación (rescate ' + fmt.num(t.rescatePct, 0) + '%)', fmt.mon(t.depKm) + '/km'],
      ['Tarifa de recuperación (gastos fijos generales)', fmt.mon(t.gfGenHora) + '/h'], ['Gastos fijos específicos', fmt.mon(t.gfEspHora) + '/h'], ['Equipamiento', fmt.mon(t.equipHora) + '/h'],
      ['Costo de capital (' + fmt.num(t.tasaCapitalPct, 1) + '% anual)', fmt.mon(t.capitalHora) + '/h'], ['Horas con cliente / vacío', fmt.num(r.tiempo.horasCliente, 2) + ' / ' + fmt.num(r.tiempo.horasVacio, 2) + ' h'],
      ['Descansos', r.descansos.n + ' × ' + fmt.mon(r.descansos.costoParada)], ['Operadores', String(r.operadores)]]
  }));
  try {
    var img = await imagenMapa(Cot.ruta.tramos);
    doc.addPage(); doc.setFont('helvetica', 'bold'); doc.setFontSize(12); doc.setTextColor(AZUL[0], AZUL[1], AZUL[2]); doc.text('Mapa de la ruta', 14, 18);
    doc.addImage(img, 'PNG', 14, 24, 188, 98);
    doc.setFont('helvetica', 'normal'); doc.setFontSize(8); doc.setTextColor(100, 100, 100); doc.text('Azul continuo: tramos con carga · Gris punteado: tramos sin carga.', 14, 128);
  } catch (e) { console.warn('Sin mapa en el PDF', e); }
  piePDF(doc, 'Desglose interno de costos. No es una cotización formal. Generado ' + fmt.fechaHora(new Date().toISOString()) + '.');
  doc.save('desglose_viaje_' + nombreArchivo(v.nombre) + '_' + hoyISO() + '.pdf');
}

/* ---------- Cotización: PDF, Excel y Word ---------- */
function datosCotizacion(c) {
  var e = D.empresa, cli = clientePorId(c.clienteId) || {}, tot = totalesCot(c), u = D.usuarios.find(function (x) { return x.id === c.usuarioId; });
  return {
    empresa_nombre: e.nombre || '', empresa_legal_name: e.legalName || '', empresa_rfc: e.rfc || '', empresa_telefono: e.telefono || '', empresa_email: e.email || '', empresa_website: e.website || '', empresa_direccion: e.direccion || '',
    COTIZACION_NUMERO: c.folio, COTIZACION_FECHA: fmt.fecha(c.fecha), COTIZACION_VIGENCIA: fmt.fecha(c.vigencia),
    cliente_nombre: cli.nombre || '', cliente_legal_name: cli.legalName || '', cliente_rfc: cli.rfc || '', cliente_contacto: cli.contacto || '', cliente_email: cli.email || '', cliente_telefono: cli.telefono || '', cliente_direccion: cli.direccion || '',
    notas: c.notas || '', elaboro: u ? u.nombre : '',
    items: (c.items || []).map(function (it, i) { return { item_n: String(i + 1), item_descripcion: it.descripcion, item_cantidad: fmt.num(it.cantidad, 0), item_unitario: fmt.mon(it.precioUnitario), item_importe: fmt.mon(num(it.cantidad) * num(it.precioUnitario)) }; }),
    subtotal: fmt.mon(tot.subtotal), IVA_PCT: fmt.num(c.ivaPct, 0) + '%', iva_monto: fmt.mon(tot.iva), total: fmt.mon(tot.total)
  };
}

function exportarCotizacionPDF(c) {
  if (!libsOk(['jspdf'])) return;
  var d = datosCotizacion(c);
  var doc = new window.jspdf.jsPDF({ unit: 'mm', format: 'letter' });
  var y = encabezadoPDF(doc, 'COTIZACIÓN', ['Cotización N.: ' + d.COTIZACION_NUMERO, 'Fecha: ' + d.COTIZACION_FECHA, 'Vigencia: ' + d.COTIZACION_VIGENCIA]);
  var w = doc.internal.pageSize.getWidth();
  doc.setFontSize(9); doc.setTextColor(30, 30, 30);
  doc.setFont('helvetica', 'bold'); doc.text('CLIENTE:', 14, y); doc.text('DATOS DEL PROVEEDOR:', w / 2 + 4, y);
  doc.setFont('helvetica', 'normal');
  var izq = [d.cliente_contacto, d.cliente_nombre, d.cliente_legal_name, d.cliente_rfc ? 'RFC: ' + d.cliente_rfc : '', d.cliente_direccion, d.cliente_email, d.cliente_telefono ? 'Tel: ' + d.cliente_telefono : ''].filter(Boolean);
  var der = [d.empresa_nombre, d.empresa_rfc ? 'RFC: ' + d.empresa_rfc : '', d.empresa_website, d.empresa_email, d.empresa_telefono ? 'Tel: ' + d.empresa_telefono : ''].filter(Boolean);
  izq.forEach(function (t, i) { doc.text(doc.splitTextToSize(t, w / 2 - 20)[0], 14, y + 5 + i * 4.2); });
  der.forEach(function (t, i) { doc.text(t, w / 2 + 4, y + 5 + i * 4.2); });
  y += 8 + Math.max(izq.length, der.length) * 4.2;
  if (d.notas) { doc.setFont('helvetica', 'bold'); doc.text('NOTAS:', 14, y); doc.setFont('helvetica', 'normal'); var ln = doc.splitTextToSize(d.notas, w - 28); doc.text(ln, 14, y + 4.5); y += 6 + ln.length * 4.2; }
  doc.autoTable(Object.assign({}, estiloTabla, {
    startY: y + 2, head: [['NO.', 'DESCRIPCIÓN', 'CANT.', 'P. UNITARIO', 'IMPORTE']],
    styles: { fontSize: 9, cellPadding: 2, lineColor: [215, 222, 228], lineWidth: 0.1 },
    columnStyles: { 0: { cellWidth: 12, halign: 'center' }, 2: { cellWidth: 18, halign: 'center' }, 3: { cellWidth: 32, halign: 'right' }, 4: { cellWidth: 34, halign: 'right' } },
    body: d.items.map(function (i) { return [i.item_n, i.item_descripcion, i.item_cantidad, i.item_unitario, i.item_importe]; })
  }));
  doc.autoTable({
    startY: doc.lastAutoTable.finalY + 2, margin: { left: w - 14 - 80 }, tableWidth: 80, theme: 'plain', styles: { fontSize: 9.5, cellPadding: 1.6 },
    columnStyles: { 0: { halign: 'right', fontStyle: 'bold' }, 1: { halign: 'right' } },
    body: [['SUBTOTAL', d.subtotal], ['IVA (' + d.IVA_PCT + ')', d.iva_monto], [{ content: 'TOTAL:', styles: { fontSize: 11 } }, { content: d.total, styles: { fontSize: 11, fontStyle: 'bold', textColor: AZUL } }]]
  });
  piePDF(doc, (d.elaboro ? 'Elaboró: ' + d.elaboro + ' · ' : '') + 'Precios en MXN. Vigencia hasta ' + d.COTIZACION_VIGENCIA + '.');
  doc.save('cotizacion_' + nombreArchivo(c.folio) + '.pdf');
}

async function exportarCotizacionExcel(c) {
  if (!libsOk(['ExcelJS'])) return;
  var d = datosCotizacion(c), tot = totalesCot(c);
  var libro = new ExcelJS.Workbook(); var h = libro.addWorksheet('Cotización');
  h.columns = [{ width: 8 }, { width: 60 }, { width: 10 }, { width: 16 }, { width: 18 }];
  h.mergeCells('A1:C1'); h.getCell('A1').value = d.empresa_nombre; h.getCell('A1').font = { bold: true, size: 14, color: { argb: 'FF0F4C81' } };
  h.mergeCells('D1:E1'); h.getCell('D1').value = 'COTIZACIÓN'; h.getCell('D1').font = { bold: true, size: 16 }; h.getCell('D1').alignment = { horizontal: 'right' };
  var info = [['Folio', d.COTIZACION_NUMERO], ['Fecha', d.COTIZACION_FECHA], ['Vigencia', d.COTIZACION_VIGENCIA], ['Cliente', d.cliente_nombre], ['RFC cliente', d.cliente_rfc], ['Contacto', d.cliente_contacto], ['Notas', d.notas]];
  info.forEach(function (f, i) { h.getCell('A' + (3 + i)).value = f[0]; h.getCell('A' + (3 + i)).font = { bold: true }; h.mergeCells('B' + (3 + i) + ':E' + (3 + i)); h.getCell('B' + (3 + i)).value = f[1]; });
  var fila = 3 + info.length + 1;
  var enc = h.getRow(fila); enc.values = ['NO.', 'DESCRIPCIÓN', 'CANT.', 'P. UNITARIO', 'IMPORTE'];
  enc.eachCell(function (cl) { cl.font = { bold: true, color: { argb: 'FFFFFFFF' } }; cl.fill = { type: 'pattern', pattern: 'solid', fgColor: { argb: 'FF0F4C81' } }; });
  (c.items || []).forEach(function (it, i) { var r = h.getRow(fila + 1 + i); r.values = [i + 1, it.descripcion, num(it.cantidad), num(it.precioUnitario), { formula: 'C' + (fila + 1 + i) + '*D' + (fila + 1 + i), result: num(it.cantidad) * num(it.precioUnitario) }]; r.getCell(4).numFmt = '$#,##0.00'; r.getCell(5).numFmt = '$#,##0.00'; });
  var f0 = fila + 1, f1 = fila + (c.items || []).length, ft = f1 + 2;
  h.getCell('D' + ft).value = 'SUBTOTAL'; h.getCell('E' + ft).value = { formula: 'SUM(E' + f0 + ':E' + f1 + ')', result: tot.subtotal };
  h.getCell('D' + (ft + 1)).value = 'IVA (' + d.IVA_PCT + ')'; h.getCell('E' + (ft + 1)).value = { formula: 'E' + ft + '*' + num(c.ivaPct) / 100, result: tot.iva };
  h.getCell('D' + (ft + 2)).value = 'TOTAL'; h.getCell('E' + (ft + 2)).value = { formula: 'E' + ft + '+E' + (ft + 1), result: tot.total };
  [ft, ft + 1, ft + 2].forEach(function (r) { h.getCell('D' + r).font = { bold: true }; h.getCell('E' + r).numFmt = '$#,##0.00'; });
  h.getCell('E' + (ft + 2)).font = { bold: true, color: { argb: 'FF0F4C81' } };
  // Hoja con el desglose de cada renglón costeado
  (c.items || []).forEach(function (it, i) {
    if (!it.snapshot || it.snapshot.tipo !== 'flete') return;
    var s = it.snapshot, r = s.resultado, hj = libro.addWorksheet('Desglose ' + (i + 1));
    hj.columns = [{ width: 40 }, { width: 20 }];
    [['Concepto', 'Importe'], ['Vehículo', s.vehiculoNombre], ['Ruta', s.rutaTexto], ['Km totales', r.kmTotal], ['Km cargados', r.kmCargado]].concat((r.componentes || []).map(function (x) { return [x.n, x.v]; }))
      .concat([['Costo total del viaje', r.costoTotalViaje], ['Gasto total', r.gastoTotal], ['Utilidad', r.utilidad], ['Financiamiento', r.financiamiento], ['Aumento por carga', r.aumentoCarga], ['Subtotal cliente', r.subtotalCliente], ['IVA', r.iva], ['Total con IVA', r.totalConIva]])
      .forEach(function (f, k) { var rw = hj.addRow(f); if (k === 0) rw.font = { bold: true }; if (typeof f[1] === 'number' && k > 4) rw.getCell(2).numFmt = '$#,##0.00'; });
  });
  var buf = await libro.xlsx.writeBuffer();
  descargarBlob(new Blob([buf], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' }), 'cotizacion_' + nombreArchivo(c.folio) + '.xlsx');
}

/* ---------- Plantillas Word (.docx) ---------- */
var VARIABLES_PLANTILLA = {
  cotizacion: [
    { g: 'Empresa (tus datos)', v: [['empresa_nombre', 'Nombre comercial.'], ['empresa_legal_name', 'Razón social.'], ['empresa_rfc', 'RFC.'], ['empresa_telefono', 'Teléfono principal.'], ['empresa_email', 'Correo de contacto.'], ['empresa_website', 'Sitio web.'], ['empresa_direccion', 'Dirección.']] },
    { g: 'Cotización', v: [['COTIZACION_NUMERO', 'Folio (ej. COT-2026-0001).'], ['COTIZACION_FECHA', 'Fecha de emisión.'], ['COTIZACION_VIGENCIA', 'Vigencia hasta.'], ['notas', 'Notas de la cotización.'], ['elaboro', 'Usuario que la elaboró.']] },
    { g: 'Cliente', v: [['cliente_nombre', 'Nombre comercial.'], ['cliente_legal_name', 'Razón social.'], ['cliente_rfc', 'RFC.'], ['cliente_contacto', 'Contacto.'], ['cliente_email', 'Correo.'], ['cliente_telefono', 'Teléfono.'], ['cliente_direccion', 'Dirección.']] },
    { g: 'Renglones (repetir fila)', v: [['#items', 'Inicio del renglón que se repite.'], ['item_n', 'Número de renglón.'], ['item_descripcion', 'Descripción.'], ['item_cantidad', 'Cantidad.'], ['item_unitario', 'Precio unitario.'], ['item_importe', 'Importe.'], ['/items', 'Fin del renglón que se repite.']] },
    { g: 'Totales', v: [['subtotal', 'Subtotal sin IVA.'], ['IVA_PCT', 'Porcentaje de IVA.'], ['iva_monto', 'Monto de IVA.'], ['total', 'Total con IVA.']] }
  ],
  precios: [
    { g: 'Empresa (tus datos)', v: [['empresa_nombre', 'Nombre comercial.'], ['empresa_legal_name', 'Razón social.'], ['empresa_rfc', 'RFC.'], ['empresa_telefono', 'Teléfono.'], ['empresa_email', 'Correo.'], ['empresa_website', 'Sitio web.']] },
    { g: 'Lista', v: [['lista_nombre', 'Nombre de la lista.'], ['lista_fecha', 'Fecha en que se fijaron los valores.'], ['ruta_nombre', 'Ruta guardada.'], ['ruta_km', 'Km de la ruta.'], ['utilidad_pct', 'Utilidad aplicada.'], ['iva_pct', 'IVA aplicado.']] },
    { g: 'Filas (repetir fila)', v: [['#filas', 'Inicio de la fila que se repite.'], ['fila_vehiculo', 'Vehículo.'], ['fila_km', 'Km.'], ['fila_tiempo', 'Tiempo de servicio.'], ['fila_peajes', 'Peajes.'], ['fila_subtotal', 'Subtotal.'], ['fila_iva', 'IVA.'], ['fila_total', 'Total con IVA.'], ['fila_por_km', 'Precio por km.'], ['fila_por_litro', 'Precio por litro.'], ['fila_por_m3', 'Precio por m³.'], ['/filas', 'Fin de la fila que se repite.']] }
  ]
};

function plantillaDefaultDocx(tipo) {
  var X = window.docx;
  var SIN = { style: X.BorderStyle.NONE, size: 0, color: 'FFFFFF' };
  var sinBordes = { top: SIN, bottom: SIN, left: SIN, right: SIN };
  var LINEA = { style: X.BorderStyle.SINGLE, size: 4, color: '999999' };
  var conBordes = { top: LINEA, bottom: LINEA, left: LINEA, right: LINEA };
  function p(texto, o) { o = o || {}; return new X.Paragraph({ alignment: o.der ? X.AlignmentType.RIGHT : o.centro ? X.AlignmentType.CENTER : X.AlignmentType.LEFT, spacing: { after: o.after == null ? 40 : o.after }, children: [new X.TextRun({ text: texto, bold: !!o.b, size: o.size || 18, color: o.color || '222222', italics: !!o.i })] }); }
  function celda(hijos, o) { o = o || {}; return new X.TableCell({ children: hijos, borders: o.bordes || sinBordes, width: o.w ? { size: o.w, type: X.WidthType.PERCENTAGE } : undefined, shading: o.fondo ? { fill: o.fondo, type: X.ShadingType.CLEAR, color: 'auto' } : undefined, margins: { top: 60, bottom: 60, left: 80, right: 80 } }); }
  function tabla(filas) { return new X.Table({ width: { size: 100, type: X.WidthType.PERCENTAGE }, rows: filas }); }
  var titulo = tipo === 'precios' ? 'LISTA DE PRECIOS' : 'COTIZACIÓN';
  var cab = tabla([new X.TableRow({ children: [
    celda([p('${empresa_nombre}', { b: true, size: 28 }), p('${empresa_legal_name}', { i: true, color: '666666' }), p('${empresa_direccion}'), p('RFC: ${empresa_rfc}'), p('Tel: ${empresa_telefono}'), p('Email: ${empresa_email}')], { w: 55 }),
    celda(tipo === 'precios' ? [p(titulo, { b: true, size: 36, der: true }), p('${lista_nombre}', { der: true, b: true }), p('Fecha: ${lista_fecha}', { der: true }), p('Ruta: ${ruta_nombre} (${ruta_km} km)', { der: true })]
      : [p(titulo, { b: true, size: 40, der: true }), p('Cotización N.: ${COTIZACION_NUMERO}', { der: true, b: true }), p('Fecha: ${COTIZACION_FECHA}', { der: true }), p('Vigencia: ${COTIZACION_VIGENCIA}', { der: true })], { w: 45 })
  ] })]);
  var hijos = [cab, p('', { after: 120 })];
  if (tipo === 'precios') {
    var enc = ['VEHÍCULO', 'KM', 'TIEMPO', 'PEAJES', 'SUBTOTAL', 'IVA', 'TOTAL', 'POR KM', 'POR LITRO'];
    var campos = ['fila_vehiculo', 'fila_km', 'fila_tiempo', 'fila_peajes', 'fila_subtotal', 'fila_iva', 'fila_total', 'fila_por_km', 'fila_por_litro'];
    hijos.push(tabla([
      new X.TableRow({ tableHeader: true, children: enc.map(function (h) { return celda([p(h, { b: true, color: 'FFFFFF', size: 16, centro: true })], { bordes: conBordes, fondo: '0F4C81' }); }) }),
      new X.TableRow({ children: campos.map(function (c, i) { return celda([p((i === 0 ? '${#filas}' : '') + '${' + c + '}' + (i === campos.length - 1 ? '${/filas}' : ''), { size: 16, der: i > 0 })], { bordes: conBordes }); }) })
    ]));
    hijos.push(p('', { after: 80 }), p('Precios en MXN. Utilidad ${utilidad_pct} · IVA ${iva_pct}. Por litro sobre el subtotal sin IVA.', { i: true, size: 16, color: '666666' }));
  } else {
    hijos.push(tabla([new X.TableRow({ children: [
      celda([p('CLIENTE:', { b: true }), p('${cliente_contacto}'), p('${cliente_nombre}'), p('RFC: ${cliente_rfc}'), p('${cliente_direccion}'), p('${cliente_email}'), p('Tel: ${cliente_telefono}')], { w: 50 }),
      celda([p('DATOS DEL PROVEEDOR:', { b: true }), p('${empresa_nombre}'), p('RFC: ${empresa_rfc}'), p('${empresa_website}'), p('${empresa_email}'), p('Tel: ${empresa_telefono}')], { w: 50 })
    ] })]));
    hijos.push(p('', { after: 80 }), p('NOTAS:', { b: true }), p('${notas}'), p('', { after: 120 }));
    var enc2 = [['NO.', 8], ['DESCRIPCIÓN', 52], ['CANT.', 10], ['P. UNITARIO', 15], ['IMPORTE', 15]];
    var campos2 = ['item_n', 'item_descripcion', 'item_cantidad', 'item_unitario', 'item_importe'];
    hijos.push(tabla([
      new X.TableRow({ tableHeader: true, children: enc2.map(function (h) { return celda([p(h[0], { b: true, color: 'FFFFFF', centro: true })], { bordes: conBordes, fondo: '0F4C81', w: h[1] }); }) }),
      new X.TableRow({ children: campos2.map(function (c, i) { return celda([p((i === 0 ? '${#items}' : '') + '${' + c + '}' + (i === campos2.length - 1 ? '${/items}' : ''), { der: i >= 3, centro: i === 0 || i === 2 })], { bordes: conBordes, w: enc2[i][1] }); }) }),
      new X.TableRow({ children: [celda([p('')], { w: 8 }), celda([p('')], { w: 52 }), celda([p('')], { w: 10 }), celda([p('SUBTOTAL', { b: true, der: true })], { w: 15 }), celda([p('${subtotal}', { der: true })], { bordes: conBordes, w: 15 })] }),
      new X.TableRow({ children: [celda([p('')], { w: 8 }), celda([p('')], { w: 52 }), celda([p('')], { w: 10 }), celda([p('IVA (${IVA_PCT})', { b: true, der: true })], { w: 15 }), celda([p('${iva_monto}', { der: true })], { bordes: conBordes, w: 15 })] }),
      new X.TableRow({ children: [celda([p('')], { w: 8 }), celda([p('')], { w: 52 }), celda([p('')], { w: 10 }), celda([p('TOTAL:', { b: true, der: true, size: 22 })], { w: 15 }), celda([p('${total}', { der: true, b: true, size: 22, color: '0F4C81' })], { bordes: conBordes, w: 15 })] })
    ]));
    hijos.push(p('', { after: 200 }), p('Elaboró: ${elaboro}', { i: true, size: 16, color: '666666' }));
  }
  var documento = new X.Document({ creator: 'Cotizador de fletes', sections: [{ properties: { page: { margin: { top: 900, bottom: 900, left: 900, right: 900 } } }, children: hijos }] });
  return X.Packer.toBlob(documento);
}

function base64ABuffer(b64) { var s = atob(b64.split(',').pop()); var u = new Uint8Array(s.length); for (var i = 0; i < s.length; i++) u[i] = s.charCodeAt(i); return u.buffer; }
async function bufferPlantilla(tipo) {
  var id = D.plantillaDefault[tipo];
  var pl = D.plantillas.find(function (x) { return x.id === id && x.tipo === tipo; });
  if (pl) return base64ABuffer(pl.base64);
  if (!libsOk(['docx'])) throw new Error('librería docx no disponible');
  return await (await plantillaDefaultDocx(tipo)).arrayBuffer();
}
function nuevoDocxtemplater(buffer) {
  return new window.docxtemplater(new window.PizZip(buffer), { delimiters: { start: '${', end: '}' }, paragraphLoop: true, linebreaks: true, nullGetter: function () { return ''; } });
}
async function generarWord(tipo, datos, nombre) {
  if (!libsOk(['PizZip', 'docxtemplater'])) return;
  try {
    var doc = nuevoDocxtemplater(await bufferPlantilla(tipo));
    doc.render(datos);
    descargarBlob(doc.getZip().generate({ type: 'blob', mimeType: 'application/vnd.openxmlformats-officedocument.wordprocessingml.document' }), nombre);
  } catch (e) {
    console.error(e);
    var det = e.properties && e.properties.errors ? e.properties.errors.map(function (x) { return x.properties && x.properties.explanation; }).filter(Boolean).slice(0, 3).join(' · ') : e.message;
    toast('No se pudo generar el Word: ' + esc(det) + '. Revisa la plantilla en el módulo Plantillas.', 'error', 8000);
  }
}
function exportarCotizacionWord(c) { generarWord('cotizacion', datosCotizacion(c), 'cotizacion_' + nombreArchivo(c.folio) + '.docx'); }

/* ---------- Tarifario: exportaciones ---------- */
function datosTarifario(t) {
  var e = D.empresa;
  return {
    empresa_nombre: e.nombre || '', empresa_legal_name: e.legalName || '', empresa_rfc: e.rfc || '', empresa_telefono: e.telefono || '', empresa_email: e.email || '', empresa_website: e.website || '', empresa_direccion: e.direccion || '',
    lista_nombre: t.nombre, lista_fecha: fmt.fecha(t.fecha), ruta_nombre: t.rutaNombre, ruta_km: fmt.num(t.rutaKm, 1), utilidad_pct: fmt.num(t.config.utilidadPct, 1) + '%', iva_pct: fmt.num(t.config.ivaPct, 0) + '%',
    filas: t.filas.map(function (f) { return { fila_vehiculo: f.vehiculoNombre, fila_km: fmt.num(f.km, 1), fila_tiempo: fmt.dur(f.horas), fila_peajes: fmt.mon(f.peajes), fila_subtotal: fmt.mon(f.subtotal), fila_iva: fmt.mon(f.iva), fila_total: fmt.mon(f.total), fila_por_km: fmt.mon(f.porKm), fila_por_litro: f.capacidadLitros ? fmt.mon(f.porLitro, 4) : '—', fila_por_m3: f.capacidadLitros ? fmt.mon(f.porM3) : '—' }; })
  };
}
function exportarTarifarioWord(t) { generarWord('precios', datosTarifario(t), 'lista_precios_' + nombreArchivo(t.nombre) + '.docx'); }
function exportarTarifarioPDF(t) {
  if (!libsOk(['jspdf'])) return;
  var d = datosTarifario(t);
  var doc = new window.jspdf.jsPDF({ unit: 'mm', format: 'letter', orientation: 'landscape' });
  var y = encabezadoPDF(doc, 'LISTA DE PRECIOS', [d.lista_nombre, 'Fecha: ' + d.lista_fecha, 'Ruta: ' + d.ruta_nombre + ' (' + d.ruta_km + ' km)']);
  doc.autoTable(Object.assign({}, estiloTabla, {
    startY: y, head: [['Vehículo', 'Km', 'Tiempo', 'Peajes', 'Subtotal', 'IVA', 'Total con IVA', 'Por km', 'Por litro', 'Por m³']],
    columnStyles: { 1: { halign: 'right' }, 2: { halign: 'right' }, 3: { halign: 'right' }, 4: { halign: 'right' }, 5: { halign: 'right' }, 6: { halign: 'right', fontStyle: 'bold' }, 7: { halign: 'right' }, 8: { halign: 'right' }, 9: { halign: 'right' } },
    body: d.filas.map(function (f) { return [f.fila_vehiculo, f.fila_km, f.fila_tiempo, f.fila_peajes, f.fila_subtotal, f.fila_iva, f.fila_total, f.fila_por_km, f.fila_por_litro, f.fila_por_m3]; })
  }));
  piePDF(doc, 'Precios en MXN con utilidad ' + d.utilidad_pct + ' e IVA ' + d.iva_pct + '. Valores fijados el ' + d.lista_fecha + '.');
  doc.save('lista_precios_' + nombreArchivo(t.nombre) + '.pdf');
}
function exportarTarifarioExcel(t) {
  exportarCatalogoExcel('Lista de precios', [
    { id: 'vehiculoNombre', titulo: 'Vehículo', anchoXls: 36 }, { id: 'km', titulo: 'Km', tipo: 'numero' }, { id: 'horas', titulo: 'Horas de servicio', tipo: 'numero' }, { id: 'peajes', titulo: 'Peajes', tipo: 'moneda' },
    { id: 'costoTotal', titulo: 'Costo del viaje', tipo: 'moneda' }, { id: 'utilidad', titulo: 'Utilidad', tipo: 'moneda' }, { id: 'subtotal', titulo: 'Subtotal', tipo: 'moneda' }, { id: 'iva', titulo: 'IVA', tipo: 'moneda' },
    { id: 'total', titulo: 'Total con IVA', tipo: 'moneda' }, { id: 'porKm', titulo: 'Por km', tipo: 'moneda' }, { id: 'porLitro', titulo: 'Por litro', tipo: 'numero' }, { id: 'porM3', titulo: 'Por m³', tipo: 'moneda' }
  ], t.filas);
}

/* ============================================================
   PLANTILLAS
   ============================================================ */
VISTAS.plantillas = function (cont) {
  var tipo = VISTAS.plantillas.tipo || 'cotizacion';
  cont.innerHTML = '<div class="pestanas"><button data-tipo="cotizacion">' + ico('fileText') + 'Cotización</button><button data-tipo="precios">' + ico('clipboard') + 'Lista de precios</button></div>' +
    '<div style="display:grid;grid-template-columns:minmax(0,1fr) 320px;gap:18px;align-items:start" class="plantillas-grid"><div><div class="fila entre" style="margin-bottom:12px"><div class="muted" style="font-size:13px">Documentos Word (.docx) para exportar ' + (tipo === 'precios' ? 'listas de precios' : 'cotizaciones') + '. Usa las variables de la derecha con el formato <span class="mono">${variable}</span>.</div><button class="btn primario" id="subirPl">' + ico('upload') + 'Subir plantilla</button></div><div id="listaPl" class="pila"></div></div>' +
    '<div class="tarjeta"><div class="cab"><span class="mono" style="color:var(--accent)">{ }</span><h3>Variables disponibles</h3></div><div class="cuerpo variables" id="varsPl"></div></div></div>';
  $$('[data-tipo]', cont).forEach(function (b) { b.classList.toggle('activa', b.dataset.tipo === tipo); b.addEventListener('click', function () { VISTAS.plantillas.tipo = b.dataset.tipo; VISTAS.plantillas(cont); }); });
  $('#varsPl').innerHTML = '<div class="faint" style="font-size:12px">Clic en una variable para copiarla.</div>' + VARIABLES_PLANTILLA[tipo].map(function (g) {
    return '<h4>' + esc(g.g) + '</h4>' + g.v.map(function (v) { return '<button data-var="${' + v[0] + '}">${' + esc(v[0]) + '}</button><small>' + esc(v[1]) + '</small>'; }).join('');
  }).join('') + '<p class="faint" style="font-size:12px;margin-top:14px">Para repetir un renglón por cada concepto, pon <span class="mono">${#' + (tipo === 'precios' ? 'filas' : 'items') + '}</span> en la primera celda del renglón y <span class="mono">${/' + (tipo === 'precios' ? 'filas' : 'items') + '}</span> en la última.</p>';
  $$('[data-var]', cont).forEach(function (b) { b.addEventListener('click', function () { (navigator.clipboard ? navigator.clipboard.writeText(b.dataset.var) : Promise.reject()).then(function () { toast('Copiado: <span class="mono">' + esc(b.dataset.var) + '</span>'); }, function () { toast('No se pudo copiar; selecciónalo manualmente.', 'warn'); }); }); });

  var nombreDefault = tipo === 'precios' ? 'plantilla-lista-precios-default' : 'plantilla-cotizacion-default';
  var items = [{ id: 'default', nombre: nombreDefault, desc: 'Plantilla de ejemplo lista para usar. Descárgala, edítala en Word y súbela como nueva.', integrada: true }]
    .concat(D.plantillas.filter(function (p) { return p.tipo === tipo; }).map(function (p) { return { id: p.id, nombre: p.nombre, desc: 'Creada: ' + fmt.fecha(p.creado) + ' · Actualizada: ' + fmt.fecha(p.actualizado), integrada: false }; }));
  $('#listaPl').innerHTML = items.map(function (it) {
    var esDef = D.plantillaDefault[tipo] === it.id;
    return '<div class="tarjeta" style="' + (esDef ? 'border-color:var(--accent)' : '') + '"><div class="cuerpo fila" style="flex-wrap:nowrap;gap:12px"><span class="icono-cab" style="width:40px;height:40px;background:var(--accent);color:#fff">' + ico('file') + '</span>' +
      '<div style="flex:1;min-width:0"><b>' + esc(it.nombre) + '</b> ' + (esDef ? '<span class="badge azul">' + ico('star') + 'Default</span>' : '') + '<div class="muted" style="font-size:12.5px">' + esc(it.desc) + '</div></div>' +
      '<button class="btn icono fantasma" data-pl-bajar="' + it.id + '" title="Descargar">' + ico('download') + '</button>' +
      (esDef ? '' : '<button class="btn icono fantasma" data-pl-def="' + it.id + '" title="Usar como default">' + ico('star') + '</button>') +
      (it.integrada ? '' : '<button class="btn icono fantasma" data-pl-reemplazar="' + it.id + '" title="Reemplazar archivo">' + ico('refresh') + '</button><button class="btn icono fantasma peligro" data-pl-borrar="' + it.id + '" title="Eliminar">' + ico('trash') + '</button>') +
      '</div></div>';
  }).join('');

  async function leerDocx(nombreSugerido) {
    var f = await elegirArchivo('.docx');
    if (!f) return null;
    if (!/\.docx$/i.test(f.name)) { toast('Solo se aceptan archivos .docx.', 'warn'); return null; }
    var dataURL = await leerArchivo(f, 'dataURL');
    try { nuevoDocxtemplater(base64ABuffer(dataURL)); }
    catch (e) { var det = e.properties && e.properties.errors ? e.properties.errors.map(function (x) { return x.properties && x.properties.explanation; }).filter(Boolean).slice(0, 2).join(' · ') : e.message; toast('La plantilla tiene errores de sintaxis: ' + esc(det), 'error', 8000); return null; }
    return { nombre: nombreSugerido || f.name.replace(/\.docx$/i, ''), base64: dataURL };
  }
  $('#subirPl').addEventListener('click', async function () {
    if (!libsOk(['PizZip', 'docxtemplater'])) return;
    var r = await leerDocx(); if (!r) return;
    var p = { id: uid('pl'), tipo: tipo, nombre: r.nombre, base64: r.base64, creado: new Date().toISOString(), actualizado: new Date().toISOString() };
    D.plantillas.push(p); D.plantillaDefault[tipo] = p.id; guardar(); toast('Plantilla subida y marcada como default.', 'ok'); VISTAS.plantillas(cont);
  });
  $('#listaPl').addEventListener('click', async function (e) {
    var b = e.target.closest('button'); if (!b) return;
    if (b.dataset.plBajar) {
      if (b.dataset.plBajar === 'default') { if (!libsOk(['docx'])) return; descargarBlob(await plantillaDefaultDocx(tipo), nombreDefault + '.docx'); }
      else { var p = D.plantillas.find(function (x) { return x.id === b.dataset.plBajar; }); descargarBlob(new Blob([base64ABuffer(p.base64)], { type: 'application/vnd.openxmlformats-officedocument.wordprocessingml.document' }), nombreArchivo(p.nombre) + '.docx'); }
    }
    if (b.dataset.plDef) { D.plantillaDefault[tipo] = b.dataset.plDef; guardar(); VISTAS.plantillas(cont); }
    if (b.dataset.plReemplazar) { var p2 = D.plantillas.find(function (x) { return x.id === b.dataset.plReemplazar; }); var r = await leerDocx(p2.nombre); if (r) { p2.base64 = r.base64; p2.actualizado = new Date().toISOString(); guardar(); toast('Plantilla reemplazada.', 'ok'); VISTAS.plantillas(cont); } }
    if (b.dataset.plBorrar && await confirmar('¿Eliminar la plantilla?', 'Eliminar', true)) { D.plantillas = D.plantillas.filter(function (x) { return x.id !== b.dataset.plBorrar; }); if (D.plantillaDefault[tipo] === b.dataset.plBorrar) D.plantillaDefault[tipo] = 'default'; guardar(); VISTAS.plantillas(cont); }
  });
};

/* ============================================================
   CONFIGURACIÓN
   ============================================================ */
VISTAS.configuracion = function (cont) {
  var e = D.empresa, f = D.folios, pa = D.parametros, rg = pa.reglaGrupo;
  function campo(ruta, etq, tipo, ayuda) { var v = getPath(D, ruta); return '<div class="campo"><label>' + etq + '</label><input type="' + (tipo || 'text') + '" step="any" data-cfg="' + ruta + '" value="' + esc(v == null ? '' : v) + '">' + (ayuda ? '<span class="ayuda">' + ayuda + '</span>' : '') + '</div>'; }
  function seccion(id, icono, titulo, cuerpo) { return '<div class="tarjeta" id="' + id + '" style="margin-bottom:14px"><div class="cab"><span class="icono-cab">' + ico(icono) + '</span><h2>' + titulo + '</h2></div><div class="cuerpo">' + cuerpo + '</div></div>'; }
  var clases = ['A2', 'C2', 'C3', 'T3-S2', 'T3-S3', 'T3-S2-R4'];
  cont.innerHTML = '<div style="max-width:1100px">' +
    seccion('cfgEmpresa', 'bank', 'Empresa', '<div class="grid-3">' + campo('empresa.nombre', 'Nombre comercial') + campo('empresa.legalName', 'Razón social') + campo('empresa.rfc', 'RFC') + campo('empresa.direccion', 'Dirección') + campo('empresa.telefono', 'Teléfono', 'tel') + campo('empresa.email', 'Correo', 'email') + campo('empresa.website', 'Sitio web', 'url') +
      '<div class="campo"><label>Logo</label><div class="fila">' + (e.logo ? '<img src="' + esc(e.logo) + '" alt="Logo" style="height:36px;border-radius:6px;background:#fff">' : '<span class="faint">Sin logo</span>') + '<button class="btn chico" id="subirLogo">' + ico('upload') + 'Subir</button>' + (e.logo ? '<button class="btn chico peligro" id="quitarLogo">Quitar</button>' : '') + '</div><span class="ayuda">PNG o JPG; aparece en el menú, PDF y Excel.</span></div></div>') +
    seccion('cfgFolios', 'hash', 'Numeración de folios', '<div class="grid-4">' + campo('folios.prefijo', 'Prefijo') +
      '<div class="campo"><label>Incluir año</label><label class="switch" style="margin-top:6px"><input type="checkbox" data-cfg-chk="folios.incluirAnio"' + (f.incluirAnio ? ' checked' : '') + '><span class="pista"></span>' + (f.incluirAnio ? 'Sí (' + new Date().getFullYear() + ')' : 'No') + '</label></div>' +
      '<div class="campo"><label>Reiniciar cada año</label><label class="switch" style="margin-top:6px"><input type="checkbox" data-cfg-chk="folios.reinicioAnual"' + (f.reinicioAnual ? ' checked' : '') + '><span class="pista"></span>' + (f.reinicioAnual ? 'Sí' : 'No') + '</label></div>' +
      '<div class="campo"><label>Próximo consecutivo</label><input type="number" min="1" id="proxFolio" value="' + (Number(siguienteFolio(true).split('-').pop())) + '"></div></div>' +
      '<div class="aviso info" style="margin-top:12px">' + ico('hash') + '<div>Folio que se generará: <b class="mono">' + esc(siguienteFolio(true)) + '</b>. Cada cotización guardada recibe un folio único y consecutivo; con el reinicio anual vuelve a 0001 al cambiar de año.</div></div>') +
    seccion('cfgParam', 'settings', 'Parámetros de cotización', '<div class="grid-3">' + campo('parametros.ivaPct', 'IVA % por defecto', 'number') + campo('parametros.utilidadPct', 'Utilidad % por defecto', 'number') + campo('parametros.financiamientoPct', 'Financiamiento % por defecto', 'number') +
      campo('parametros.vigenciaDias', 'Vigencia por defecto (días)', 'number') + campo('parametros.factorTrafico', 'Factor de tiempo «con tráfico»', 'number', 'Multiplica el tiempo de manejo del ruteo. 1.15 = +15%. Supuesto editable: calíbralo con tus tiempos reales.') + campo('parametros.velMaxDefault', 'Vel. máx. por defecto (km/h)', 'number') + '</div>') +
    seccion('cfgRuteo', 'route', 'Ruteo y mapas', '<div class="grid-2"><div class="campo"><label>Proveedor de ruteo</label><select data-cfg-sel="parametros.ruteo.proveedor"><option value="osrm"' + (pa.ruteo.proveedor !== 'ors' ? ' selected' : '') + '>OSRM público (sin llave · perfil automóvil)</option><option value="ors"' + (pa.ruteo.proveedor === 'ors' ? ' selected' : '') + '>OpenRouteService (perfil camión · requiere API key)</option></select></div>' +
      '<div class="campo"><label>API key de OpenRouteService</label><input type="password" data-cfg="parametros.ruteo.orsApiKey" value="' + esc(pa.ruteo.orsApiKey || '') + '" autocomplete="off"><span class="ayuda">Se guarda en los datos del cotizador en texto plano; usa una llave de uso restringido.</span></div></div>' +
      '<div class="aviso" style="margin-top:12px">' + ico('alert') + '<div>El servidor público de OSRM calcula rutas de <b>automóvil</b> y su política de uso no es para producción. Para rutas de camión (restricciones de peso/altura) usa OpenRouteService o un servidor propio. Las casetas no se detectan solas: se calculan con tu catálogo de casetas (abajo) o se capturan en cada ruta.</div></div>') +
    seccion('cfgDir', 'database', 'Directorio de estaciones y TAD', '<div class="grid-2">' + campo('parametros.directorioCsv', 'Archivo CSV junto al HTML') + '<div class="campo"><label>Estado</label><div class="fila" style="margin-top:4px">' + (Directorio.cargado ? '<span class="badge verde">' + fmt.ent(Directorio.estaciones.length) + ' estaciones · ' + fmt.ent(Directorio.terminales.length) + ' TAD</span>' : '<span class="badge ambar">No cargado</span>') + '<button class="btn chico" id="cargarCsv">' + ico('upload') + 'Cargar CSV manualmente</button><button class="btn chico" id="reintentarCsv">' + ico('refresh') + 'Reintentar</button></div></div></div>' +
      '<p class="faint" style="font-size:12px;margin:8px 0 0">Columnas esperadas: pv.Permiso, pv.Latitud_Y, pv.Longitud_X, r.TAD, r.LATITUD, r.LONGUITUD.</p>') +
    seccion('cfgRegla', 'users', 'Regla comercial del grupo', '<div class="grid-4"><div class="campo"><label>Activa</label><label class="switch" style="margin-top:6px"><input type="checkbox" data-cfg-chk="parametros.reglaGrupo.activa"' + (rg.activa ? ' checked' : '') + '><span class="pista"></span>' + (rg.activa ? 'Sí' : 'No') + '</label></div>' +
      campo('parametros.reglaGrupo.tarifaM3', 'Tarifa fija $/m³', 'number') + campo('parametros.reglaGrupo.limiteKm', 'Hasta (km de ruta)', 'number') + campo('parametros.reglaGrupo.utilidadRutaLarga', 'Utilidad % arriba del límite', 'number') + '</div>' +
      '<div class="aviso" style="margin-top:12px">' + ico('alert') + '<div>Aplica solo a clientes tipo «Grupo Servifácil» y a vehículos con capacidad en litros. Tarifa y límite vienen del cotizador anterior (la página mostraba $110.00/m³ pero calculaba con $170.77/m³): <b>deben validarlos Comercial y Finanzas</b> antes de usarse.</div></div>') +
    seccion('cfgCarga', 'box', 'Tipos de carga', '<div id="tablaCarga"></div><button class="btn chico" id="nuevaCarga" style="margin-top:8px">' + ico('plus') + 'Agregar tipo</button>') +
    seccion('cfgCasetas', 'dollar', 'Catálogo de casetas', '<div class="aviso info" style="margin-bottom:10px">' + ico('info') + '<div>Registra cada caseta con su ubicación y tarifa por clase de vehículo. Si la ruta pasa a menos de 350 m de una caseta, se suma su tarifa según el tipo del vehículo. Verifica las tarifas vigentes con el operador de la autopista (CAPUFE u otro).</div></div><div class="barra-acciones"><button class="btn chico" id="nuevaCaseta">' + ico('plus') + 'Agregar caseta</button><button class="btn chico" id="impCasetas">' + ico('upload') + 'Importar</button><button class="btn chico" id="expCasetas">' + ico('download') + 'Exportar</button></div><div id="tablaCasetas"></div>') +
    seccion('cfgZonas', 'alert', 'Zonas restringidas', '<p class="muted" style="margin-top:0;font-size:13px">Si una ruta entra a una zona, el cotizador prueba las rutas alternas configuradas y usa la más corta.</p><div id="tablaZonas"></div>') +
    seccion('cfgRespaldo', 'save', 'Datos y respaldo', '<p style="margin-top:0">Modo actual: <b>' + ({ servidor: 'servidor (guardado automático)', archivo: 'archivo datos_cotizador.json (solo lectura)', local: 'datos de ejemplo en memoria' }[Persistencia.modo]) + '</b>.</p>' +
      '<div class="fila"><button class="btn" id="bajarRespaldo">' + ico('download') + 'Descargar respaldo (JSON)</button><button class="btn" id="subirRespaldo">' + ico('upload') + 'Importar respaldo</button><button class="btn peligro" id="restablecer">' + ico('refresh') + 'Restablecer datos de ejemplo</button></div>' +
      '<p class="faint" style="font-size:12px">Sin servidor, los cambios viven solo en esta pestaña. Para guardarlos de forma permanente ejecuta <span class="mono">node servidor.js</span> en la carpeta del HTML, o descarga el respaldo y colócalo junto al HTML como <span class="mono">datos_cotizador.json</span>.</p>') +
    '</div>';

  cont.querySelector('div').addEventListener('change', function (ev) {
    var el = ev.target;
    if (el.dataset.cfg) { setPath(D, el.dataset.cfg, el.type === 'number' ? num(el.value) : el.value); guardar(); if (/^empresa\.nombre|^folios\./.test(el.dataset.cfg)) { pintarNav(); VISTAS.configuracion(cont); } if (el.dataset.cfg === 'parametros.directorioCsv') { Directorio.intentado = false; } }
    if (el.dataset.cfgChk) { setPath(D, el.dataset.cfgChk, el.checked); guardar(); VISTAS.configuracion(cont); }
    if (el.dataset.cfgSel) { setPath(D, el.dataset.cfgSel, el.value); guardar(); }
    if (el.id === 'proxFolio') { var n2 = Math.max(1, Math.round(num(el.value))); D.folios.consecutivo = n2 - 1; D.folios.anio = new Date().getFullYear(); guardar(); VISTAS.configuracion(cont); }
  });
  $('#subirLogo').addEventListener('click', async function () { var f2 = await elegirArchivo('image/png,image/jpeg'); if (!f2) return; if (f2.size > 600000) { toast('El logo pesa más de 600 KB; usa una imagen más ligera.', 'warn'); return; } D.empresa.logo = await leerArchivo(f2, 'dataURL'); guardar(); pintarNav(); VISTAS.configuracion(cont); });
  var ql = $('#quitarLogo'); if (ql) ql.addEventListener('click', function () { D.empresa.logo = ''; guardar(); pintarNav(); VISTAS.configuracion(cont); });
  $('#cargarCsv').addEventListener('click', async function () { var f3 = await elegirArchivo('.csv,text/csv'); if (!f3) return; try { cargarDirectorioTexto(await leerArchivo(f3, 'texto')); Directorio.intentado = true; toast('Directorio cargado: ' + Directorio.estaciones.length + ' estaciones y ' + Directorio.terminales.length + ' TAD.', 'ok'); VISTAS.configuracion(cont); } catch (err) { toast(esc(err.message), 'error'); } });
  $('#reintentarCsv').addEventListener('click', async function () { Directorio.intentado = false; await cargarDirectorio(); toast(Directorio.cargado ? 'Directorio cargado.' : Directorio.error, Directorio.cargado ? 'ok' : 'warn', 6000); VISTAS.configuracion(cont); });
  $('#bajarRespaldo').addEventListener('click', exportarRespaldo);
  $('#subirRespaldo').addEventListener('click', importarRespaldo);
  $('#restablecer').addEventListener('click', async function () { if (await confirmar('Se borrarán <b>todos</b> los catálogos, rutas y cotizaciones y se cargarán los datos de ejemplo.', 'Restablecer', true)) { D = datosIniciales(); Cot = null; guardar(); pintarNav(); navegar(); } });

  var tc = tablaCatalogo({ cont: $('#tablaCarga'), filas: function () { return D.tiposCarga; }, columnas: [{ id: 'nombre', titulo: 'Tipo de carga' }, { id: 'aumentoPct', titulo: 'Aumento %', tipo: 'numero', dec: 1 }], alCambiar: function (x, k, v) { x[k] = v; },
    acciones: function () { return [{ txt: 'Eliminar', icono: 'trash', peligro: true, fn: function (x) { if (D.tiposCarga.length <= 1) { toast('Debe quedar al menos un tipo.', 'warn'); return; } D.tiposCarga = D.tiposCarga.filter(function (y) { return y !== x; }); guardar(); tc.pintar(); } }]; } });
  $('#nuevaCarga').addEventListener('click', function () { D.tiposCarga.push({ id: uid('tc'), nombre: 'Nuevo tipo', aumentoPct: 0 }); guardar(); tc.pintar(); });

  var colsCaseta = [{ id: 'nombre', titulo: 'Caseta' }, { id: 'lat', titulo: 'Latitud', tipo: 'numero', dec: 5 }, { id: 'lon', titulo: 'Longitud', tipo: 'numero', dec: 5 }].concat(clases.map(function (k) {
    return { id: 't_' + k, titulo: k, tipo: 'moneda', valor: function (c) { return num((c.tarifas || {})[k]); }, exportar: function (c) { return num((c.tarifas || {})[k]); } };
  }));
  var tcas = tablaCatalogo({ cont: $('#tablaCasetas'), filas: function () { return D.casetas; }, columnas: colsCaseta,
    alCambiar: function (c, k, v) { if (k.indexOf('t_') === 0) { c.tarifas = c.tarifas || {}; c.tarifas[k.slice(2)] = num(v); } else c[k] = k === 'nombre' ? v : num(v); },
    acciones: function () { return [{ txt: 'Eliminar', icono: 'trash', peligro: true, fn: function (c) { D.casetas = D.casetas.filter(function (x) { return x !== c; }); guardar(); tcas.pintar(); } }]; },
    vacio: 'Sin casetas registradas. Mientras tanto, captura el total de peajes en cada ruta.' });
  $('#nuevaCaseta').addEventListener('click', function () { D.casetas.push({ id: uid('cas'), nombre: 'Nueva caseta', lat: 0, lon: 0, tarifas: {} }); guardar(); tcas.pintar(); toast('Doble clic en cada celda para editarla.'); });
  $('#expCasetas').addEventListener('click', function () { exportarCatalogoExcel('Casetas', colsCaseta, D.casetas); });
  $('#impCasetas').addEventListener('click', async function () { await importarCatalogo(colsCaseta.map(function (c) { return Object.assign({}, c, { tipo: c.id === 'nombre' ? undefined : 'numero' }); }), function (o) { var c = { id: uid('cas'), nombre: o.nombre || 'Caseta', lat: num(o.lat), lon: num(o.lon), tarifas: {} }; clases.forEach(function (k) { if (o['t_' + k] != null) c.tarifas[k] = num(o['t_' + k]); }); D.casetas.push(c); }); tcas.pintar(); });

  var tz = tablaCatalogo({ cont: $('#tablaZonas'), filas: function () { return D.zonasRestringidas; }, columnas: [
    { id: 'nombre', titulo: 'Zona' }, { id: 'lat', titulo: 'Latitud centro', tipo: 'numero', dec: 5 }, { id: 'lon', titulo: 'Longitud centro', tipo: 'numero', dec: 5 }, { id: 'radioKm', titulo: 'Radio (km)', tipo: 'numero', dec: 1 },
    { id: 'alt', titulo: 'Rutas alternas', editable: false, render: function (z) { return (z.alternas || []).map(function (a) { return esc(a.nombre); }).join('<br>') || '<span class="faint">Ninguna</span>'; } }
  ], alCambiar: function (z, k, v) { z[k] = k === 'nombre' ? v : num(v); },
    acciones: function () { return [{ txt: 'Eliminar', icono: 'trash', peligro: true, fn: function (z) { D.zonasRestringidas = D.zonasRestringidas.filter(function (x) { return x !== z; }); guardar(); tz.pintar(); } }]; } });
};

/* ============================================================
   USUARIOS Y ROLES
   ============================================================ */
VISTAS.usuarios = function (cont) {
  var tab = VISTAS.usuarios.tab || 'usuarios';
  cont.innerHTML = '<div class="pestanas"><button data-tab="usuarios">' + ico('users') + 'Usuarios</button><button data-tab="roles">' + ico('shield') + 'Roles y permisos</button></div>' +
    '<div class="aviso" style="margin-bottom:12px">' + ico('lock') + '<div>Los roles controlan qué módulos se muestran en esta página. <b>No sustituyen la autenticación:</b> para restringir el acceso real se requiere inicio de sesión en el servidor (validar con TI y Seguridad).</div></div><div id="usrCuerpo"></div>';
  $$('[data-tab]', cont).forEach(function (b) { b.classList.toggle('activa', b.dataset.tab === tab); b.addEventListener('click', function () { VISTAS.usuarios.tab = b.dataset.tab; VISTAS.usuarios(cont); }); });
  var c = $('#usrCuerpo');
  if (tab === 'usuarios') {
    c.innerHTML = '<div class="barra-acciones"><button class="btn primario" id="nuevoUsr">' + ico('plus') + 'Nuevo usuario</button></div><div id="tablaUsr"></div>';
    var t = tablaCatalogo({ cont: $('#tablaUsr'), filas: function () { return D.usuarios; }, columnas: [
      { id: 'foto', titulo: 'Foto', editable: false, ordenable: false, render: function (u) { return '<span class="avatar">' + esc((u.nombre || '?').charAt(0)) + '</span>'; } },
      { id: 'nombre', titulo: 'Nombre' }, { id: 'email', titulo: 'Email' }, { id: 'puesto', titulo: 'Puesto de trabajo' },
      { id: 'rolId', titulo: 'Rol', tipo: 'select', opciones: function () { return D.roles.map(function (r) { return { v: r.id, t: r.nombre }; }); } },
      { id: 'estatus', titulo: 'Estatus', tipo: 'select', opciones: [{ v: 'alta', t: 'Alta' }, { v: 'baja', t: 'Baja' }] }
    ], alCambiar: function (u, k, v) {
      if (k === 'rolId' && u.rolId === 'admin' && v !== 'admin' && D.usuarios.filter(function (x) { return x.rolId === 'admin' && x.estatus !== 'baja'; }).length <= 1) { toast('Debe quedar al menos un administrador.', 'warn'); return; }
      u[k] = v; pintarNav();
    },
    acciones: function () { return [{ txt: 'Eliminar', icono: 'trash', peligro: true, fn: function (u) { if (u.id === D.usuarioActivoId) { toast('No puedes eliminar al usuario activo.', 'warn'); return; } D.usuarios = D.usuarios.filter(function (x) { return x !== u; }); guardar(); t.pintar(); } }]; } });
    $('#nuevoUsr').addEventListener('click', function () { D.usuarios.push({ id: uid('u'), nombre: 'Nuevo usuario', email: '', puesto: '', rolId: 'usuario', estatus: 'alta' }); guardar(); t.pintar(); toast('Doble clic en cada celda para editarla.'); });
    return;
  }
  var sel = VISTAS.usuarios.rol || null;
  var rol = D.roles.find(function (r) { return r.id === sel; });
  c.innerHTML = '<div style="display:grid;grid-template-columns:260px 1fr;gap:14px;align-items:start"><div class="tarjeta"><div class="cab"><h3>Roles</h3><div style="flex:1"></div><button class="btn chico suave" id="nuevoRol">' + ico('plus') + 'Rol</button></div><div class="cuerpo" style="padding:8px">' +
    D.roles.map(function (r) { return '<button class="btn bloque' + (r.id === sel ? ' suave' : '') + '" style="justify-content:flex-start;margin-bottom:4px;border-color:transparent" data-rol="' + r.id + '">' + ico('lock') + esc(r.nombre) + (r.sistema ? '<span class="badge" style="margin-left:auto">Sistema</span>' : '') + '</button>'; }).join('') +
    '</div></div><div class="tarjeta"><div class="cuerpo" id="permisos">' + (rol ? '' : '<div class="vacio-estado">Selecciona un rol para ver o editar sus permisos.</div>') + '</div></div></div>';
  if (rol) {
    var todos = rol.permisos.indexOf('*') >= 0;
    $('#permisos').innerHTML = '<div class="fila entre"><h3 style="margin:0">' + esc(rol.nombre) + '</h3>' + (rol.sistema ? '<span class="badge">Rol del sistema · solo lectura</span>' : '<button class="btn chico peligro" id="borrarRol">' + ico('trash') + 'Eliminar rol</button>') + '</div>' +
      (rol.sistema ? '' : '<div class="campo" style="max-width:300px;margin-top:10px"><label>Nombre del rol</label><input type="text" id="nombreRol" value="' + esc(rol.nombre) + '"></div>') +
      '<div class="grid-3" style="margin-top:12px">' + MODULOS.map(function (m) { return '<label class="fila" style="padding:6px 0"><input type="checkbox" data-perm="' + m.id + '"' + (todos || rol.permisos.indexOf(m.id) >= 0 ? ' checked' : '') + (rol.sistema ? ' disabled' : '') + '>' + ico(m.icono) + esc(m.nombre) + '</label>'; }).join('') + '</div>';
    $$('[data-perm]', c).forEach(function (x) { x.addEventListener('change', function () { rol.permisos = $$('[data-perm]:checked', c).map(function (y) { return y.dataset.perm; }); guardar(); pintarNav(); }); });
    var nr = $('#nombreRol'); if (nr) nr.addEventListener('change', function () { rol.nombre = nr.value; guardar(); VISTAS.usuarios(cont); });
    var br = $('#borrarRol'); if (br) br.addEventListener('click', function () { if (D.usuarios.some(function (u) { return u.rolId === rol.id; })) { toast('Hay usuarios con este rol; reasígnalos primero.', 'warn'); return; } D.roles = D.roles.filter(function (r) { return r !== rol; }); VISTAS.usuarios.rol = null; guardar(); VISTAS.usuarios(cont); });
  }
  $$('[data-rol]', c).forEach(function (b) { b.addEventListener('click', function () { VISTAS.usuarios.rol = b.dataset.rol; VISTAS.usuarios(cont); }); });
  $('#nuevoRol').addEventListener('click', function () { var r = { id: uid('rol'), nombre: 'Nuevo rol', sistema: false, permisos: ['inicio', 'cotizar'] }; D.roles.push(r); VISTAS.usuarios.rol = r.id; guardar(); VISTAS.usuarios(cont); });
};

/* ============================================================
   DOCUMENTACIÓN DEL SISTEMA
   ============================================================ */
VISTAS.documentacion = function (cont) {
  var toc = [
    ['doc-vision', 'Visión general', 1], ['doc-flujo', 'Flujo de cotización', 1], ['doc-conceptos', 'Conceptos clave', 1],
    ['doc-catalogos', 'Catálogos: tus datos', 1], ['doc-perfil', 'Perfil del vehículo', 2], ['doc-equipos', 'Equipamiento', 2], ['doc-gastos', 'Gastos fijos generales', 2], ['doc-clientes', 'Clientes', 2], ['doc-listado', 'Cotizaciones: listado', 2],
    ['doc-calculos', 'Cálculos de costos', 1], ['doc-c-tiempo', 'Tiempo de servicio', 2], ['doc-c-comb', 'Combustible', 2], ['doc-c-neum', 'Neumáticos', 2], ['doc-c-mtto', 'Mantenimiento', 2], ['doc-c-desc', 'Descansos', 2], ['doc-c-dep', 'Depreciación', 2], ['doc-c-cap', 'Costo de capital', 2], ['doc-c-gf', 'Gastos fijos', 2], ['doc-c-eq', 'Equipamiento', 2], ['doc-c-pago', 'Pago del operador', 2],
    ['doc-vacios', 'Cliente vs. tramos vacíos', 1], ['doc-precio', 'Del costo al precio', 1], ['doc-estrategias', 'Estrategias de cobro', 1], ['doc-renta', 'Renta / disposición', 1],
    ['doc-rutas', 'Rutas y peajes', 1], ['doc-tarifario', 'Lista de precios (tarifario)', 1], ['doc-folios', 'Numeración de folios', 1], ['doc-ejemplo', 'Ejemplo de validación', 1], ['doc-datos', 'Datos, respaldo y servidor', 1], ['doc-supuestos', 'Supuestos por validar', 1]
  ];
  var tr = Motor.tarifaRecuperacion(D);
  cont.innerHTML = '<div class="docs"><nav><div class="titulo">Contenido</div>' + toc.map(function (t) { return '<a href="#' + t[0] + '" class="n' + t[2] + '" data-ir="' + t[0] + '">' + esc(t[1]) + '</a>'; }).join('') + '</nav><article>' +
    '<h2 id="doc-vision">' + ico('target') + 'Documentación del sistema</h2><p class="sub">Qué toma en cuenta el cotizador para calcular el costo real de un viaje, cómo llega al precio de venta y dónde se captura cada dato.</p>' +
    '<p>Es una herramienta de <b>costeo y cotización para autotransporte</b>, adaptada al traslado de petrolíferos en autotanque. A partir del perfil de costos de cada vehículo y de la ruta trazada en el mapa, calcula el costo real de operar el viaje y lo convierte en un precio de venta con utilidad, financiamiento e IVA.</p>' +
    '<div class="tarjetas-doc"><div><b>' + ico('box') + ' Costeo por componentes</b>Cada rubro (combustible, llantas, depreciación…) se calcula por separado con los datos del vehículo, así que todo precio es auditable rubro por rubro.</div><div><b>' + ico('split') + ' Cliente vs. tramos vacíos</b>El costo de los kilómetros con carga se separa del posicionamiento y regresos en vacío; la utilidad se aplica, por defecto, solo a la parte productiva.</div><div><b>' + ico('clock') + ' Datos actuales y precios fijados</b>Cotizar usa siempre los datos actuales. Al guardar una cotización o tarifario se conserva una copia del cálculo que reproduce el precio histórico.</div></div>' +
    '<h3>Módulos</h3><div class="tabla-wrap"><table class="tabla"><tbody>' +
    [['Cotizar servicio', 'Mapa interactivo: traza la ruta, separa tramos con y sin carga y calcula el costo del viaje en vivo. También cotiza renta de unidades.'], ['Cotizaciones', 'Historial con folio, estados, desglose guardado y exportación a Word, PDF y Excel.'], ['Lista de precios', 'Tarifario ruta × vehículo, guardable y recalculable a datos de hoy.'], ['Vehículos', 'Perfiles de costo: rendimiento, llantas, depreciación, gastos propios y pago del operador.'], ['Equipamiento', 'Equipo de abordo con precio; cada vehículo indica piezas y frecuencia de cambio.'], ['Gastos fijos generales', 'Gastos anuales de la empresa y tiempo efectivo de la flota (tarifa de recuperación).'], ['Clientes', 'Directorio para cotizar; el tipo «Grupo» activa la regla comercial interna.'], ['Plantillas', 'Documentos Word (.docx) con variables para cotizaciones y listas de precios.'], ['Configuración', 'Datos de la empresa, folios, parámetros, ruteo, casetas, zonas restringidas y respaldo.'], ['Usuarios y roles', 'Quién cotiza y qué módulos ve cada rol.']]
      .map(function (f) { return '<tr><td style="width:190px"><b>' + f[0] + '</b></td><td>' + f[1] + '</td></tr>'; }).join('') + '</tbody></table></div>' +

    '<h2 id="doc-flujo">' + ico('route') + 'Flujo de una cotización</h2><p class="sub">De la selección del vehículo al folio guardado, en seis pasos.</p><ol class="pasos">' +
    '<li><b>Selección del vehículo.</b> Se cargan sus parámetros: rendimiento, llantas, mantenimiento, depreciación, gastos fijos de su categoría, equipamiento y esquema de pago del operador.</li>' +
    '<li><b>Trazado de la ruta.</b> Las paradas se eligen del directorio de estaciones y TAD, se buscan en el mapa o se marcan con un clic. Se calcula la distancia y el tiempo de cada tramo; cada parada define si el tramo que sale de ella va con carga o vacío.</li>' +
    '<li><b>Peajes.</b> Se suman las casetas de tu catálogo por las que pasa la ruta, con la tarifa de la clase del vehículo, o se captura el total.</li>' +
    '<li><b>Análisis de costos.</b> Se calcula cada rubro y se separa en «con utilidad» (cliente) y «tramos vacíos».</li>' +
    '<li><b>Precio de venta.</b> Se suman la utilidad, el financiamiento (si aplica), el aumento por tipo de carga y el IVA.</li>' +
    '<li><b>Guardado con folio.</b> La cotización recibe un folio consecutivo (ej. COT-2026-0001) y guarda una copia del cálculo para reproducir el precio después.</li></ol>' +

    '<h2 id="doc-conceptos">' + ico('bulb') + 'Conceptos clave</h2><div class="tarjetas-doc">' +
    '<div><b>Kilómetros cargados y vacíos</b>Cada tramo hereda la marca «con carga» de la parada donde inicia. Los km cargados generan valor para el cliente; los vacíos son posicionamiento o regreso.</div>' +
    '<div><b>Horas con cliente vs. sin utilidad</b>Esperas, descansos obligatorios y recargas cuentan como tiempo del cliente, porque la unidad está dedicada a su servicio. Solo el manejo sin carga es tiempo vacío.</div>' +
    '<div><b>Costo por hora</b>Los gastos fijos, el equipamiento y el costo de capital se convierten en un costo por cada hora que trabaja la flota; cada viaje recupera las horas que ocupa.</div></div>' +
    '<div class="tarjetas-doc" style="grid-template-columns:1fr 1fr"><div><b>Copia del cálculo</b>Cotizaciones y tarifarios guardan la ruta, los parámetros y las decisiones de negocio. Se puede consultar el precio original o recalcular «a datos de hoy».</div><div><b>Precio por litro y por m³</b>Para autotanques, el subtotal (sin IVA) se divide entre la capacidad del perfil: es la referencia para comparar contra el costo logístico por litro.</div></div>' +

    '<h2 id="doc-catalogos">' + ico('database') + 'Catálogos: tus datos base</h2><p>Antes de cotizar, el sistema necesita conocer tu operación. La calidad de la cotización depende de qué tan fieles a la realidad sean estos datos.</p>' +
    '<h3 id="doc-perfil">Perfil del vehículo</h3><div class="tabla-wrap"><table class="tabla"><thead><tr><th class="sin-orden">Sección</th><th class="sin-orden">Datos</th><th class="sin-orden">Para qué sirve</th></tr></thead><tbody>' +
    [['Identificación', 'Plantilla, nombre, tipo (A2, C2, C3, T3-S2, T3-S3, T3-S2-R4), capacidad en litros', 'El tipo pre-llena llantas, pide activos requeridos y define la clase para casetas.'], ['Dimensiones y peso', 'Alto, ancho, largo, peso, capacidad de remolque', 'Validar la ruta y clasificar la unidad.'], ['Valor', 'Valor, vida útil (km y años), tasa de capital, valor de rescate', 'Depreciación y costo de capital.'], ['Neumáticos', 'Direccionales, motrices y arrastre: cantidad, vida útil y precio', 'Desgaste por kilómetro.'], ['Combustible', 'Km/L vacío y cargado, precio/L, tanque, recarga, ralentí', 'Litros y costo del viaje; tiempo de recarga.'], ['Descanso', 'Frecuencia, duración, costo por parada', 'Paradas obligatorias en viajes largos.'], ['Mantenimiento', 'Intervalo y costo del preventivo', 'Costo por kilómetro.'], ['Activos requeridos', 'Remolques, tanques, dollies: valor y vida útil', 'Se deprecian igual que el vehículo.'], ['Gastos fijos del vehículo', 'Categoría y gastos propios (seguro, GPS, permisos)', 'Costo por hora de la unidad.'], ['Equipamiento', 'Equipo del catálogo, cantidad y frecuencia de cambio', 'Costo por día y por hora.'], ['Pago del operador', 'Operadores, $/km, $/hora', 'Valores de partida al cotizar.']]
      .map(function (f) { return '<tr><td><b>' + f[0] + '</b></td><td>' + f[1] + '</td><td>' + f[2] + '</td></tr>'; }).join('') + '</tbody></table></div>' +
    '<div class="aviso" style="margin-top:10px">' + ico('alert') + '<div><b>Los perfiles de ejemplo son un punto de partida.</b> El tractocamión T3-S3 reproduce el ejemplo del manual de referencia y los autotanques copian esos valores. Ajústalos a tus unidades reales antes de cotizar; sin categoría de vehículo, los gastos fijos y el equipamiento quedan en cero.</div></div>' +
    '<h3 id="doc-equipos">Equipamiento</h3><p>Catálogo del equipo de abordo (extintor, gato, bandas, cadenas…). El precio se captura <b>sin IVA</b>; cada vehículo indica cuántas piezas lleva y cada cuánto las reemplaza.</p>' +
    '<h3 id="doc-gastos">Gastos fijos generales</h3><p>Lo que pagas aunque no salga ningún viaje: renta, administración, sueldos base, seguros, permisos. Se anualizan y se reparten entre las horas facturables de toda la flota:</p>' +
    '<div class="formula">Tarifa de recuperación ($/h) = Σ gastos fijos anualizados ÷ Σ (vehículos × días/año × horas/día)\nHoy: ' + fmt.mon(tr.gastoAnual) + ' ÷ ' + fmt.ent(tr.horasFlota) + ' h = ' + fmt.mon(tr.tarifaHora) + '/h</div>' +
    '<p>Usa <b>horas realmente facturables</b>: sobrestimarlas baja la tarifa y deja gastos sin recuperar.</p>' +
    '<h3 id="doc-clientes">Clientes</h3><p>Nombre, razón social, RFC, contacto y dirección se usan en las exportaciones. El tipo «Grupo Servifácil» activa la regla comercial interna configurable.</p>' +
    '<h3 id="doc-listado">Cotizaciones: listado</h3><p>Estados: <b>Borrador</b>, <b>Enviada</b>, <b>Aceptada</b>, <b>Rechazada</b>; una cotización en borrador o enviada pasa a <b>Vencida</b> cuando la fecha de vigencia queda atrás. El Inicio resume el mes: emitidas, monto sin impuestos, tasa de cierre (aceptadas ÷ emitidas) y lo que está por vencer.</p>' +

    '<h2 id="doc-calculos">' + ico('chart') + 'Cálculos de costos</h2><p class="sub">Qué toma en cuenta cada rubro y de dónde sale el dato. En el detalle del viaje ves cuánto suma cada uno; con clic en el Total con IVA se descarga el PDF con el desglose completo.</p>' +
    '<h3 id="doc-c-tiempo">Tiempo de servicio</h3><div class="formula">Servicio = manejo × factor de tráfico (mín. km ÷ vel. máx.) + paradas programadas + descansos + recargas\nHoras con cliente = manejo cargado + paradas + descansos + recargas\nHoras vacío = manejo sin carga</div><p>Las esperas en carga y descarga también cuestan: la unidad no está disponible para otro servicio.</p>' +
    '<h3 id="doc-c-comb">Combustible</h3><div class="formula">Litros = km cargados ÷ km/L cargado + km vacíos ÷ km/L vacío + horas de ralentí × L/h\nCosto = litros × precio por litro</div><p>El rendimiento y el ralentí vienen del perfil; el precio y los rendimientos se pueden ajustar en cada cotización.</p>' +
    '<h3 id="doc-c-neum">Neumáticos</h3><div class="formula">$/km = Σ (cantidad × precio ÷ vida útil km) de direccionales, motrices y arrastre</div>' +
    '<h3 id="doc-c-mtto">Mantenimiento</h3><div class="formula">$/km = costo del servicio preventivo ÷ intervalo en km</div>' +
    '<h3 id="doc-c-desc">Descansos del operador</h3><div class="formula">Descansos = redondeo hacia arriba (horas de manejo ÷ frecuencia) − 1\nCosto = descansos × costo por parada; tiempo = descansos × duración</div>' +
    '<h3 id="doc-c-dep">Depreciación</h3><div class="formula">$/km = valor × (1 − % rescate) ÷ vida útil km   (vehículo y cada activo requerido)</div><p>Lo que esperas recuperar al vender la unidad no se cobra.</p>' +
    '<h3 id="doc-c-cap">Costo de capital</h3><div class="formula">Capital promedio = (Σ valores + Σ valores × % rescate) ÷ 2\n$/h = capital promedio × tasa anual ÷ horas/año de la categoría</div><p>Depende del tiempo que la unidad está dedicada al viaje, no de los kilómetros. Déjalo en 0 si el pago del crédito ya está en gastos fijos: no lo cobres dos veces.</p>' +
    '<h3 id="doc-c-gf">Gastos fijos</h3><div class="formula">Generales = tarifa de recuperación × horas\nEspecíficos = Σ gastos del vehículo anualizados ÷ horas/año de su categoría × horas</div>' +
    '<h3 id="doc-c-eq">Equipamiento</h3><div class="formula">Anual = Σ precio × cantidad ÷ (años + meses ÷ 12)\n$/día = anual ÷ días/año · $/h = anual ÷ horas/año</div>' +
    '<h3 id="doc-c-pago">Pago del operador</h3><p>Solo lo que se le paga <b>por cada viaje</b> (el sueldo base va en gastos fijos). Se pueden combinar: por kilómetro (con y sin carga), por hora (con y sin carga) y comisión (% sobre el neto o sobre la utilidad). Las comisiones salen de la utilidad y no suben el precio al cliente.</p>' +

    '<h2 id="doc-vacios">' + ico('split') + 'Cliente vs. tramos vacíos</h2><p>El sistema separa el costo en dos columnas antes de calcular la ganancia:</p><ul><li>Lo que gasta la unidad en tramos con carga va a la columna del cliente; lo de los tramos sin carga, a tramos vacíos.</li><li>Las esperas en paradas y los descansos van completos a la columna del cliente.</li><li>Las casetas se cobran tal cual, sin utilidad.</li></ul>' +
    '<p>En el detalle del viaje, cada rubro vacío tiene dos casillas: <b>Cobrar</b> (azul) y <b>Utilidad</b> (amarillo). Lo no cobrado lo absorbe la empresa.</p>' +

    '<h2 id="doc-precio">' + ico('dollar') + 'Del costo al precio</h2><ol class="pasos"><li><b>Gasto total del viaje:</b> columna del cliente + tramos vacíos cobrados + casetas + pago al operador + otros gastos.</li><li><b>Utilidad:</b> porcentaje sobre la parte con utilidad (cliente, vacíos marcados, pago al operador si está marcado y otros gastos marcados).</li><li><b>Financiamiento:</b> porcentaje opcional sobre el gasto total (crédito al cliente).</li><li><b>Aumento por tipo de carga:</b> porcentaje sobre el gasto total (por ejemplo, material peligroso).</li><li><b>IVA:</b> sobre el subtotal cliente.</li></ol>' +
    '<div class="formula">Subtotal cliente = gasto total + utilidad + financiamiento + aumento por carga\nTotal con IVA = subtotal cliente × (1 + IVA%)\nPor km = total con IVA ÷ km de la ruta · Por tonelada = total con IVA ÷ toneladas\nPor litro / m³ = subtotal cliente ÷ capacidad del autotanque</div>' +

    '<h2 id="doc-estrategias">' + ico('target') + 'Estrategias de cobro</h2><p>Un viaje real tiene hasta tres tramos: <b>posicionamiento</b> (de la base a donde se carga, en vacío), <b>servicio</b> (lo que el cliente reconoce) y <b>retorno</b> (de vuelta a la base). Los tres consumen diésel, llantas y horas.</p>' +
    '<p>La decisión que define el precio: <b>¿cobras el retorno?</b> Si no tienes carga de regreso asegurada, inclúyelo («Trazar regreso»). Quitarlo es apostar a conseguir carga de regreso: decídelo viaje por viaje.</p>' +
    '<p><b>Vende un servicio, no kilómetros.</b> Presenta precio por viaje (o por tonelada / por litro) y no por kilómetro cargado, que delata los tramos vacíos. El desglose es tu <b>piso de costo</b>: nunca cotices por debajo. Si el viaje es casi todo vacío, marca rubros vacíos con utilidad o cobra el día de servicio.</p>' +

    '<h2 id="doc-renta">' + ico('calendar') + 'Renta / disposición</h2><p>Cobro por tener la unidad a disposición del cliente, no por viaje. Unidades: por hora, jornada, día, semana, mensual (1/12 del año de operación) y renta fija (precio cerrado sugerido desde el costo).</p>' +
    '<div class="formula">Disponibilidad = horas × (gastos fijos generales + específicos + equipamiento + capital)/h\n              + depreciación anual × fracción del año + sueldo de operador × operadores × meses\nUso = km × (combustible si va incluido + neumáticos + mantenimiento) + casetas si van incluidas\nPrecio = (disponibilidad + uso) × (1 + utilidad%) · sin cargo de financiamiento</div>' +
    '<p>La jornada sale del Tiempo efectivo de la categoría del vehículo. Con un tope de km, el excedente se cobra con recargo (50% por defecto) por km y por hora; el sistema avisa si el uso estimado lo supera. Para estimar el uso, traza una ruta representativa y define la frecuencia de viajes.</p>' +

    '<h2 id="doc-rutas">' + ico('map') + 'Rutas y peajes</h2><p>La distancia y el tiempo de cada tramo vienen del proveedor de ruteo configurado: <b>OSRM público</b> (perfil automóvil, sin llave) u <b>OpenRouteService</b> (perfil camión, requiere API key). El tiempo se ajusta con el factor de tráfico y la velocidad máxima.</p>' +
    '<p>Si una ruta atraviesa una <b>zona restringida</b> (por ejemplo, el poblado de Chiltepec), se prueban las rutas alternas configuradas y se usa la más corta.</p>' +
    '<p>Las casetas se detectan con <b>tu catálogo de casetas</b> (ubicación y tarifa por clase de vehículo). Si no hay catálogo, captura el total en «Peajes». Verifica las tarifas vigentes con el operador de cada autopista.</p>' +

    '<h2 id="doc-tarifario">' + ico('clipboard') + 'Lista de precios (tarifario)</h2><p>Aplica el mismo costeo a una ruta guardada con varios vehículos. Cada lista pasa por tres estados: <b>vista en vivo</b> (se calcula con los parámetros actuales), <b>guardada con valores fijados</b> (copia del cálculo) y <b>recalculada a datos de hoy</b> (misma estrategia con el combustible, rendimiento y tarifas actuales). Los valores de la barra se aplican a todas las filas respetando el rendimiento de cada vehículo. Se exporta a Excel, PDF y Word.</p>' +

    '<h2 id="doc-folios">' + ico('hash') + 'Numeración de folios</h2><p>Se configura en Configuración: prefijo, si incluye el año y si el consecutivo se reinicia cada año.</p><div class="formula">Prefijo COT · incluye año 2026 · consecutivo 1  →  COT-2026-0001</div>' +

    '<h2 id="doc-ejemplo">' + ico('checkCircle') + 'Ejemplo de validación</h2><p>Con el perfil «Tractocamión caja seca (T3-S3)» de ejemplo, 65.113 km cargados, 49 min de manejo, peajes de $543.00, utilidad 20% e IVA 16%, el motor reproduce el ejemplo del manual de referencia:</p>' +
    '<div class="tabla-wrap"><table class="tabla"><tbody>' + [['Combustible (34.27 L × $23.41)', '$802.26'], ['Neumáticos ($0.57/km)', '$37.03'], ['Mantenimiento ($0.40/km)', '$26.05'], ['Depreciación (vehículo $1.57/km + remolque $0.44/km)', '$130.85'], ['Gastos fijos generales ($86.18/h)', '$70.49'], ['Gastos fijos específicos ($40.15/h)', '$32.84'], ['Equipamiento ($6.88/h)', '$5.62'], ['Costo de capital ($46.24/h)', '$37.82'], ['Subtotal con utilidad', '$1,142.97'], ['+ Peajes + pago al operador ($3/km)', '$543.00 + $195.34'], ['Costo total del viaje', '$1,881.30'], ['Utilidad 20%', '$267.66'], ['Subtotal cliente', '$2,148.97'], ['IVA 16%', '$343.83'], ['Total con IVA', '$2,492.80']].map(function (f) { return '<tr><td>' + f[0] + '</td><td class="num"><b>' + f[1] + '</b></td></tr>'; }).join('') + '</tbody></table></div>' +

    '<h2 id="doc-datos">' + ico('save') + 'Datos, respaldo y servidor</h2><ul><li><b>Con servidor</b> (<span class="mono">node servidor.js</span> en la carpeta del HTML): los cambios se guardan solos en <span class="mono">datos_cotizador.json</span> y se comparten entre usuarios de la red.</li><li><b>Sin servidor</b>: si existe <span class="mono">datos_cotizador.json</span> junto al HTML se carga al abrir; los cambios viven en la pestaña hasta que descargues el respaldo.</li><li>El directorio de estaciones y TAD se lee del CSV junto al HTML o se carga manualmente en Configuración.</li></ul>' +

    '<h2 id="doc-supuestos">' + ico('alert') + 'Supuestos por validar</h2><div class="aviso">' + ico('alert') + '<div><ul style="margin:0;padding-left:18px">' +
    '<li>Los catálogos de ejemplo (gastos fijos, equipamiento, perfiles) provienen del manual de referencia y no son datos de Servifácil.</li>' +
    '<li>Factor de tráfico (1.15) y velocidad máxima: calibrar contra tiempos reales de la flota.</li>' +
    '<li>Regla comercial del grupo ($170.77/m³ hasta 100 km; 20% arriba): heredada del cotizador anterior, pendiente de validación por Comercial y Finanzas.</li>' +
    '<li>Aumento por tipo de carga y financiamiento se calculan sobre el gasto total.</li>' +
    '<li>Anualización de gastos: diario × 365, semanal × 52, quincenal × 24, mensual × 12.</li>' +
    '<li>IVA, tarifas de casetas y precio del diésel cambian: verificarlos contra fuentes oficiales vigentes. Los aspectos fiscales requieren revisión del área Fiscal.</li></ul></div></div>' +
    '</article></div>';
  $$('[data-ir]', cont).forEach(function (a) { a.addEventListener('click', function (e) { e.preventDefault(); var s = document.getElementById(a.dataset.ir); if (s) s.scrollIntoView({ behavior: 'smooth', block: 'start' }); }); });
};

/* ============================================================
   ARRANQUE
   ============================================================ */
(async function iniciar() {
  try { await cargarDatos(); }
  catch (e) { console.error(e); D = datosIniciales(); Persistencia.modo = 'local'; }
  $('#btnColapsar').innerHTML = ico('panel');
  $('#btnColapsar').addEventListener('click', function () { $('#app').classList.toggle('colapsada'); if (mapa) setTimeout(function () { mapa.invalidateSize(); }, 220); });
  $('#btnTema').innerHTML = ico('sun');
  $('#btnTema').addEventListener('click', function () {
    var claro = document.documentElement.getAttribute('data-theme') !== 'light';
    document.documentElement.setAttribute('data-theme', claro ? 'light' : 'dark');
    $('#btnTema').innerHTML = ico(claro ? 'moon' : 'sun');
  });
  $('#btnUsuario').addEventListener('click', menuUsuario);
  $('#estadoGuardado').addEventListener('click', function () { if ($('#estadoGuardado').classList.contains('pendiente')) exportarRespaldo(); });
  window.addEventListener('hashchange', navegar);
  window.addEventListener('beforeunload', function (e) { if (Persistencia.pendiente && Persistencia.modo !== 'servidor') { e.preventDefault(); e.returnValue = ''; } });
  pintarNav(); pintarEstadoGuardado(); navegar();
  if (Persistencia.modo === 'local') toast('Trabajando con <b>datos de ejemplo</b> en memoria. Para guardar de forma permanente ejecuta <span class="mono">servidor.js</span> o descarga el respaldo.', 'warn', 8000);
})();

</script>
</body>
</html>