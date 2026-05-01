# Guardar el contenido del dashboard.py en un archivo
%%writefile dashboard.py
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import streamlit as st

# Configuración de página
st.set_page_config(page_title="Dashboard de Ventas", layout="wide", page_icon="📊")

# Título y descripción
st.title("📊 Dashboard Interactivo de Ventas Online")
st.markdown("---")

# Cargar datos
@st.cache_data
def load_data():
    df = pd.read_csv("/content/drive/MyDrive/Online Sales Data.csv") # Corrected file path
    df['Date'] = pd.to_datetime(df['Date'])
    df['Month'] = df['Date'].dt.to_period('M').astype(str)
    df['Year'] = df['Date'].dt.year
    return df

df = load_data()

# Sidebar con filtros
st.sidebar.header("🔍 Filtros")

# Filtro por categoría
categorias = st.sidebar.multiselect(
    "Categoría de Producto",
    options=df['Product Category'].unique(),
    default=df['Product Category'].unique()
)

# Filtro por región
regiones = st.sidebar.multiselect(
    "Región",
    options=df['Region'].unique(),
    default=df['Region'].unique()
)

# Filtro por método de pago
pagos = st.sidebar.multiselect(
    "Método de Pago",
    options=df['Payment Method'].unique(),
    default=df['Payment Method'].unique()
)

# Filtro por rango de fechas
fecha_min = df['Date'].min().date()
fecha_max = df['Date'].max().date()
rango_fechas = st.sidebar.date_input(
    "Rango de Fechas",
    value=(fecha_min, fecha_max),
    min_value=fecha_min,
    max_value=fecha_max
)

# Aplicar filtros
df_filtrado = df[
    (df['Product Category'].isin(categorias)) &
    (df['Region'].isin(regiones)) &
    (df['Payment Method'].isin(pagos)) &
    (df['Date'].dt.date >= rango_fechas[0]) &
    (df['Date'].dt.date <= rango_fechas[1])
]

# KPIs principales
kpi1, kpi2, kpi3, kpi4 = st.columns(4)
with kpi1:
    st.metric("💰 Ingresos Totales", f"${df_filtrado['Total Revenue'].sum():,.2f}")
with kpi2:
    st.metric("📦 Unidades Vendidas", f"{df_filtrado['Units Sold'].sum():,}")
with kpi3:
    st.metric("🧾 Transacciones", f"{len(df_filtrado):,}")
with kpi4:
    st.metric("🎫 Ticket Promedio", f"${df_filtrado['Total Revenue'].mean():.2f}")

st.markdown("--- Jardiel---")

# Fila 1: Gráficos principales
col1, col2 = st.columns(2)

with col1:
    st.subheader("📈 Ingresos por Categoría")
    cat_rev = df_filtrado.groupby('Product Category')['Total Revenue'].sum().reset_index()
    fig_cat = px.pie(cat_rev, values='Total Revenue', names='Product Category',
                     hole=0.4, color_discrete_sequence=px.colors.sequential.Bluered)
    st.plotly_chart(fig_cat, width='stretch')

with col2:
    st.subheader("🌍 Ingresos por Región")
    region_rev = df_filtrado.groupby('Region')['Total Revenue'].sum().reset_index()
    fig_region = px.bar(region_rev, x='Region', y='Total Revenue',
                        color='Region', text_auto='.2s')
    st.plotly_chart(fig_region, width='stretch')

# Fila 2: Tendencias y métodos de pago
col3, col4 = st.columns(2)

with col3:
    st.subheader("📅 Tendencia Mensual de Ventas")
    monthly_rev = df_filtrado.groupby('Month')['Total Revenue'].sum().reset_index()
    fig_trend = px.line(monthly_rev, x='Month', y='Total Revenue',
                        markers=True, line_shape='spline')
    fig_trend.update_traces(line_color='#2E86AB', marker_size=8)
    st.plotly_chart(fig_trend, width='stretch')

with col4:
    st.subheader("💳 Métodos de Pago")
    payment_rev = df_filtrado.groupby('Payment Method')['Total Revenue'].sum().reset_index()
    fig_payment = px.pie(payment_rev, values='Total Revenue', names='Payment Method',
                           hole=0.4, color_discrete_sequence=px.colors.qualitative.Pastel) # Changed px.donut to px.pie with hole
    st.plotly_chart(fig_payment, width='stretch')

# Fila 3: Top productos y análisis regional
col5, col6 = st.columns(2)

with col5:
    st.subheader("🏆 Top 10 Productos por Ingresos")
    top_products = df_filtrado.groupby('Product Name')['Total Revenue'].sum().nlargest(10).reset_index()
    fig_top = px.bar(top_products, x='Total Revenue', y='Product Name',
                     orientation='h', color='Total Revenue',
                     color_continuous_scale='Viridis')
    fig_top.update_layout(yaxis={'categoryorder':'total ascending'})
    st.plotly_chart(fig_top, width='stretch')

with col6:
    st.subheader("🗺️ Ventas por Región y Categoría")
    pivot_data = df_filtrado.groupby(['Region', 'Product Category'])['Total Revenue'].sum().reset_index()
    fig_heat = px.sunburst(pivot_data, path=['Region', 'Product Category'],
                           values='Total Revenue', color='Total Revenue',
                           color_continuous_scale='RdBu')
    st.plotly_chart(fig_heat, width='stretch')

# Fila 4: Tabla de datos y análisis adicional
with st.expander("📋 Ver Datos Detallados"):
    st.dataframe(df_filtrado.style.format({
        'Unit Price': '${:.2f}',
        'Total Revenue': '${:.2f}'
    }), width='stretch') # Changed use_container_width to width='stretch'

# Análisis adicional
st.markdown("### 🔍 Insights Adicionales")
col7, col8 = st.columns(2)

with col7:
    st.subheader("📊 Unidades Vendidas por Categoría")
    cat_units = df_filtrado.groupby('Product Category')['Units Sold'].sum().reset_index()
    fig_units = px.bar(cat_units, x='Product Category', y='Units Sold',
                       color='Units Sold', color_continuous_scale='YlOrRd')
    st.plotly_chart(fig_units, width='stretch')

with col8:
    st.subheader("💵 Precio Promedio por Categoría")
    cat_price = df_filtrado.groupby('Product Category')['Unit Price'].mean().reset_index()
    fig_price = px.scatter(cat_price, x='Product Category', y='Unit Price',
                           size='Unit Price', color='Product Category',
                           size_max=60)
    st.plotly_chart(fig_price, width='stretch')

# Footer
st.markdown("--- Jardiel---")
st.caption("📊 Dashboard generado con Streamlit y Plotly | Datos actualizados hasta Agosto 2024")