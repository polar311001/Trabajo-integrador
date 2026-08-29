# Trabajo-integrador
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_excel("/content/ModeloComercial.xlsx")

df

# CARGAR TODAS LAS TABLAS
ventas = pd.read_excel("/content/ModeloComercial.xlsx", sheet_name='Ventas')
pais = pd.read_excel("/content/ModeloComercial.xlsx", sheet_name='País')
cliente = pd.read_excel("/content/ModeloComercial.xlsx", sheet_name='Cliente')
vendedor = pd.read_excel("/content/ModeloComercial.xlsx", sheet_name='Vendedor')
operador = pd.read_excel("/content/ModeloComercial.xlsx", sheet_name='Operador')
marca = pd.read_excel("/content/ModeloComercial.xlsx", sheet_name='Marca')
tienda = pd.read_excel("/content/ModeloComercial.xlsx", sheet_name='Tienda')
modelo = pd.read_excel("/content/ModeloComercial.xlsx", sheet_name='Modelo')

# CONOCER LA ESTRUCTURA DE LOS DATOS

ventas.head()
ventas.tail()
ventas.shape()
ventas.columns
ventas.info()
ventas.describe()

# Para revisar cada tabla
print("Ventas:", ventas.shape)
print("País:", pais.shape)
print("Cliente:", cliente.shape)
print("Vendedor:", vendedor.shape)
print("Operador:", operador.shape)
print("Marca:", marca.shape)
print("Tienda:", tienda.shape)
print("Modelo:", modelo.shape)

# Revisar valores nulos

ventas.isnull().sum()
(ventas.isnull().sum() / len(ventas) * 100).sort_values(ascending=False)

# Revisar todos los nulos para todas las tablas

for nombre, df in {
    'Ventas': ventas,
    'País': pais,
    'Cliente': cliente,
    'Vendedor': vendedor,
    'Operador': operador,
    'Marca': marca,
    'Tienda': tienda,
    'Modelo': modelo
}.items():
    print(f"\n--- {nombre} ---")
    print(df.isnull().sum())

# revisar duplicados

ventas.duplicated().sum()

# ver los duplicados

ventas[ventas.duplicated()]
ventas['Factura'].duplicated().sum()

# revisar tipos de datos

ventas.dtypes
ventas['Fecha'] = pd.to_datetime(ventas['Fecha'])
ventas['FechaVencimiento'] = pd.to_datetime(ventas['FechaVencimiento'])
ventas['FechaPago'] = pd.to_datetime(ventas['FechaPago'])

# CREACION DE VARIABLES PARA EL ANALISIS

ventas['Año'] = ventas['Fecha'].dt.year
ventas['Mes'] = ventas['Fecha'].dt.month
ventas['NombreMes'] = ventas['Fecha'].dt.month_name()
ventas['Trimestre'] = ventas['Fecha'].dt.quarter
ventas['DiaSemana'] = ventas['Fecha'].dt.day_name()

# CREACION DE VARIABLES ADICIONALES

ventas['CostoTotal'] = ventas['Cantidad'] * ventas['Costo']
ventas['VentaTotal'] = ventas['Cantidad'] * ventas['Precio']
ventas['Venta'].equals(ventas['VentaTotal'])
ventas['Utilidad'] = ventas['Venta'] - ventas['CostoTotal']
ventas['Margen'] = ventas['Utilidad'] / ventas['Venta'] * 100

# CREACION DE KPIS

ventas_totales = ventas['Venta'].sum()
costos_totales = ventas['CostoTotal'].sum()
utilidad_total = ventas['Utilidad'].sum()
unidades_vendidas = ventas['Cantidad'].sum()
numero_facturas = ventas['Factura'].nunique()
margen_promedio = ventas['Margen'].mean()

print(f"Ventas totales: ${ventas_totales:,.2f}")
print(f"Costos totales: ${costos_totales:,.2f}")
print(f"Utilidad total: ${utilidad_total:,.2f}")
print(f"Unidades vendidas: {unidades_vendidas:,}")
print(f"Número de facturas: {numero_facturas:,}")
print(f"Margen promedio: {margen_promedio:.2f}%")

# VENTAS POR AÑO 

ventas_año = ventas.groupby('Año').agg(
    Ventas=('Venta', 'sum'),
    Unidades=('Cantidad', 'sum'),
    Utilidad=('Utilidad', 'sum')
).reset_index()

ventas_año

# CRECIMIENTO ANUAL

ventas_año['CrecimientoVentas_%'] = ventas_año['Ventas'].pct_change() * 100

ventas_año

# GRAFICO DE EVOLUCION DE VENTAS

plt.figure(figsize=(12,6))

plt.plot(
    ventas_año['Año'],
    ventas_año['Ventas'],
    marker='o'
)

plt.title('Evolución de las ventas por año')
plt.xlabel('Año')
plt.ylabel('Ventas')
plt.grid(True)

plt.show()

# VENTAS MENSUALES

ventas_mes = ventas.groupby(
    ventas['Fecha'].dt.to_period('M')
)['Venta'].sum().reset_index()

ventas_mes['Fecha'] = ventas_mes['Fecha'].astype(str)

ventas_mes



plt.figure(figsize=(15,6))

plt.plot(
    ventas_mes['Fecha'],
    ventas_mes['Venta'],
    marker='o'
)

plt.title('Evolución mensual de las ventas')
plt.xlabel('Mes')
plt.ylabel('Ventas')
plt.xticks(rotation=90)

plt.show()

# VENTAS POR PAIS

ventas_pais = ventas.merge(
    pais,
    on='CódigoPaís',
    how='left'
)

ventas_por_pais = ventas_pais.groupby(
    'NombrePaís'
).agg(
    Ventas=('Venta', 'sum'),
    Unidades=('Cantidad', 'sum'),
    Utilidad=('Utilidad', 'sum')
).sort_values(
    'Ventas',
    ascending=False
)

ventas_por_pais

# ventas por 10 paises

ventas_por_pais.head(10)

top10 = ventas_por_pais.head(10)

plt.figure(figsize=(12,6))

plt.barh(
    top10.index[::-1],
    top10['Ventas'][::-1]
)

plt.title('Top 10 países por ventas')
plt.xlabel('Ventas')
plt.ylabel('País')

plt.show()

# PARTICIPACION DE CADA PAIS

ventas_por_pais['Participacion_%'] = (
    ventas_por_pais['Ventas'] /
    ventas_por_pais['Ventas'].sum()
) * 100

ventas_por_pais

# VENTAS POR MARCA

ventas_modelo = ventas.merge(
    modelo,
    on='CódigoModelo',
    how='left',
    suffixes=('_Venta', '_Modelo')
)
ventas_marca = ventas_modelo.merge(
    marca,
    on='CódigoMarca',
    how='left'
)

ventas_por_marca = ventas_marca.groupby(
    'DescripciónMarca'
).agg(
    Ventas=('Venta', 'sum'),
    Unidades=('Cantidad', 'sum'),
    Utilidad=('Utilidad', 'sum')
).sort_values(
    'Ventas',
    ascending=False
)

ventas_por_marca

# # grafico de ventas por marca
plt.figure(figsize=(10,6))

plt.bar(
    ventas_por_marca.index,
    ventas_por_marca['Ventas']
)

plt.title('Ventas por marca')
plt.xlabel('Marca')
plt.ylabel('Ventas')
plt.xticks(rotation=45)

plt.show()

# VENTAS POR MODELO
ventas_por_modelo = ventas_modelo.groupby(
    'DescripciónModelo'
).agg(
    Ventas=('Venta', 'sum'),
    Unidades=('Cantidad', 'sum'),
    Utilidad=('Utilidad', 'sum')
).sort_values(
    'Ventas',
    ascending=False
)
ventas_por_modelo.head(10)
top_modelos = ventas_por_modelo.head(10)

plt.figure(figsize=(12,6))

plt.barh(
    top_modelos.index[::-1],
    top_modelos['Ventas'][::-1]
)

plt.title('Top 10 modelos por ventas')
plt.xlabel('Ventas')

plt.show()

# ANALISIS DE VENDEDORES
ventas_vendedor = ventas.merge(
    vendedor,
    on='CódigoVendedor',
    how='left'
)
ventas_por_vendedor = ventas_vendedor.groupby(
    'NombreVendedor'
).agg(
    Ventas=('Venta', 'sum'),
    Unidades=('Cantidad', 'sum'),
    Utilidad=('Utilidad', 'sum'),
    Facturas=('Factura', 'nunique')
).sort_values(
    'Ventas',
    ascending=False
)

ventas_por_vendedor

# ANALIZAR TICKET PROMEDIO 

ticket_promedio = ventas['Venta'].sum() / ventas['Factura'].nunique()

print(f"Ticket promedio: ${ticket_promedio:,.2f}")

ventas_por_vendedor['TicketPromedio'] = (
    ventas_por_vendedor['Ventas'] /
    ventas_por_vendedor['Facturas']
)

ventas_por_vendedor

# ANALISIS DE TIENDAS
ventas_tienda = ventas.merge(
    tienda,
    on='CódigoTienda',
    how='left'
)
ventas_por_tienda = ventas_tienda.groupby(
    'RazónSocialTienda'
).agg(
    Ventas=('Venta', 'sum'),
    Unidades=('Cantidad', 'sum'),
    Utilidad=('Utilidad', 'sum')
).sort_values(
    'Ventas',
    ascending=False
)

ventas_por_tienda
plt.figure(figsize=(10,6))

plt.bar(
    ventas_por_tienda.index,
    ventas_por_tienda['Ventas']
)

plt.title('Ventas por tienda')
plt.xlabel('Tienda')
plt.ylabel('Ventas')
plt.xticks(rotation=45)

plt.show()

# ANALISIS POR OPERADOR
ventas_operador = ventas.merge(
    operador,
    on='CódigoOperador',
    how='left'
)
ventas_por_operador = ventas_operador.groupby(
    'RazónSocialOperador'
).agg(
    Ventas=('Venta', 'sum'),
    Unidades=('Cantidad', 'sum'),
    Utilidad=('Utilidad', 'sum')
).sort_values(
    'Ventas',
    ascending=False
)

ventas_por_operador

# ANALISIS DE CLIENTES
ventas_cliente = ventas.merge(
    cliente,
    on='CódigoCliente',
    how='left'
)

ventas_por_cliente = ventas_cliente.groupby(
    'RazónSocial'
).agg(
    Ventas=('Venta', 'sum'),
    Unidades=('Cantidad', 'sum'),
    Utilidad=('Utilidad', 'sum'),
    Facturas=('Factura', 'nunique')
).sort_values(
    'Ventas',
    ascending=False
)

ventas_por_cliente
ventas_por_cliente.head(10)

# ANALISIS DE RENTABILIDAD

rentabilidad_modelo = ventas_modelo.groupby(
    'DescripciónModelo'
).agg(
    Ventas=('Venta', 'sum'),
    Costos=('CostoTotal', 'sum'),
    Utilidad=('Utilidad', 'sum')
)

rentabilidad_modelo['Margen_%'] = (
    rentabilidad_modelo['Utilidad'] /
    rentabilidad_modelo['Ventas']
) * 100

rentabilidad_modelo.sort_values(
    'Utilidad',
    ascending=False
).head(10)

# PRODUCTO CON MAYOR MARGEN

rentabilidad_modelo.sort_values(
    'Margen_%',
    ascending=False
).head(10)

# PRODUCTOS CON MENOR MARGEN

rentabilidad_modelo.sort_values(
    'Margen_%'
).head(10)

# RELACION ENTRE CANTIDAD Y VENTAS

plt.figure(figsize=(10,6))

plt.scatter(
    ventas['Cantidad'],
    ventas['Venta'],
    alpha=0.5
)

plt.title('Relación entre cantidad vendida y ventas')
plt.xlabel('Cantidad')
plt.ylabel('Ventas')

plt.show()

# MATRIZ DE CORRELACION

variables = [
    'Cantidad',
    'Costo',
    'Precio',
    'Venta',
    'CostoTotal',
    'Utilidad',
    'Margen'
]

correlacion = ventas[variables].corr()

correlacion

plt.figure(figsize=(10,7))

sns.heatmap(
    correlacion,
    annot=True,
    cmap='coolwarm',
    fmt='.2f'
)

plt.title('Matriz de correlación')

plt.show()

# ANALISIS DE PAGOS

ventas['DiasPago'] = (
    ventas['FechaPago'] -
    ventas['FechaVencimiento']
).dt.days
ventas['DiasPago'].describe()

# IDENTIFICAR PAGOS ATRASADOS
ventas['PagoAtrasado'] = ventas['DiasPago'] > 0
ventas['PagoAtrasado'].value_counts()
ventas['PagoAtrasado'].mean() * 100

# DIAS PROMEDIO DE ATRASO

atrasos = ventas[ventas['DiasPago'] > 0]

atrasos['DiasPago'].mean()

# CLIENTES QUE MAS SE ATRASAN

atrasos_cliente = ventas.merge(
    cliente,
    on='CódigoCliente',
    how='left'
)

atrasos_cliente.groupby(
    'RazónSocial'
).agg(
    DiasAtrasoPromedio=('DiasPago', 'mean'),
    OperacionesAtrasadas=('Factura', 'count')
).sort_values(
    'DiasAtrasoPromedio',
    ascending=False
)

# DASHBOARD DE KPIS EN PYTHON

fig, axes = plt.subplots(2, 2, figsize=(15,10))

# Ventas por año
axes[0,0].plot(
    ventas_año['Año'],
    ventas_año['Ventas'],
    marker='o'
)
axes[0,0].set_title('Ventas por año')

# Ventas por marca
axes[0,1].bar(
    ventas_por_marca.index,
    ventas_por_marca['Ventas']
)
axes[0,1].set_title('Ventas por marca')
axes[0,1].tick_params(axis='x', rotation=45)

# Ventas por país
top5 = ventas_por_pais.head(5)
axes[1,0].barh(
    top5.index[::-1],
    top5['Ventas'][::-1]
)
axes[1,0].set_title('Top 5 países')

# Ventas por vendedor
axes[1,1].bar(
    ventas_por_vendedor.index,
    ventas_por_vendedor['Ventas']
)
axes[1,1].set_title('Ventas por vendedor')

plt.tight_layout()
plt.show()

# TABLA FINAL DE KPIS

kpis = pd.DataFrame({
    'Indicador': [
        'Ventas totales',
        'Costos totales',
        'Utilidad total',
        'Unidades vendidas',
        'Número de facturas',
        'Ticket promedio',
        'Margen promedio',
        'Días promedio de pago'
    ],
    'Valor': [
        ventas['Venta'].sum(),
        ventas['CostoTotal'].sum(),
        ventas['Utilidad'].sum(),
        ventas['Cantidad'].sum(),
        ventas['Factura'].nunique(),
        ticket_promedio,
        ventas['Margen'].mean(),
        ventas['DiasPago'].mean()
    ]
})

kpis

# ANALISIS ABC DE PRODUCTOS

abc = ventas_modelo.groupby(
    'DescripciónModelo'
)['Venta'].sum().sort_values(
    ascending=False
).reset_index()

abc['VentasAcumuladas'] = abc['Venta'].cumsum()

abc['PorcentajeAcumulado'] = (
    abc['VentasAcumuladas'] /
    abc['Venta'].sum()
) * 100

abc['CategoriaABC'] = np.where(
    abc['PorcentajeAcumulado'] <= 80,
    'A',
    np.where(
        abc['PorcentajeAcumulado'] <= 95,
        'B',
        'C'
    )
)

abc

# ENCONTRAR EL PRODUCTO ESTRELLA

producto_estrella = ventas_por_modelo.sort_values(
    'Ventas',
    ascending=False
).head(1)

producto_estrella

# ENCONTRAR EL PAIS LIDER
vendedor_lider = ventas_por_vendedor.head(1)

vendedor_lider

# ENCONTRAR EL VENDEDOR LIDER
vendedor_lider = ventas_por_vendedor.head(1)

vendedor_lider

# ENCONTRAR LA MARCA LIDER
marca_lider = ventas_por_marca.head(1)

marca_lider

# ANALISIS DE VENTAS + RENTABILIDAD

analisis_final = ventas_modelo.groupby(
    ['DescripciónMarca', 'DescripciónModelo']
).agg(
    Unidades=('Cantidad', 'sum'),
    Ventas=('Venta', 'sum'),
    Costos=('CostoTotal', 'sum'),
    Utilidad=('Utilidad', 'sum')
).reset_index()

analisis_final['Margen_%'] = (
    analisis_final['Utilidad'] /
    analisis_final['Ventas']
) * 100

analisis_final = analisis_final.sort_values(
    'Ventas',
    ascending=False
)

analisis_final.head(20)

# EXPORTAR TODOS LOS RESULTADOS A EXCEL
with pd.ExcelWriter('Analisis_Ventas.xlsx') as writer:

    kpis.to_excel(writer, sheet_name='KPIs', index=False)
    ventas_año.to_excel(writer, sheet_name='Ventas_Año', index=False)
    ventas_por_pais.to_excel(writer, sheet_name='Ventas_Pais')
    ventas_por_marca.to_excel(writer, sheet_name='Ventas_Marca')
    ventas_por_modelo.to_excel(writer, sheet_name='Ventas_Modelo')
    ventas_por_vendedor.to_excel(writer, sheet_name='Ventas_Vendedor')
    ventas_por_tienda.to_excel(writer, sheet_name='Ventas_Tienda')
    ventas_por_cliente.to_excel(writer, sheet_name='Ventas_Cliente')
    ventas_por_operador.to_excel(writer, sheet_name='Ventas_Operador')
    analisis_final.to_excel(writer, sheet_name='Analisis_Final', index=False)

