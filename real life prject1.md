<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# i would be making it in 2 hours and in 1 script either locally or on databricks

give the outine for notebook

Perfect! Here's a **2-hour, single-notebook project outline** that's impressive yet achievable:

***

## 📓 **ExcelOps: 2-Hour Sales Processing Pipeline**

### **Notebook Structure (Time-boxed)**

```python
# Cell 1: Header & Setup (5 min)
"""
ExcelOps: Automated Sales Processing Engine
==========================================
Input: Messy Excel file with sales data
Output: Clean multi-sheet Excel report with insights

Pipeline Steps:
1. Extract & Validate (20 min)
2. Clean & Transform (30 min)
3. Business Logic & KPIs (25 min)
4. Generate Reports (30 min)
5. Testing & Demo (10 min)

Total: 2 hours
"""
```


***

## 🗂️ **Detailed Outline**

### **Section 1: Setup \& Configuration (5 minutes)**

```python
# 📦 Imports
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
import warnings
warnings.filterwarnings('ignore')

# For Excel formatting
from openpyxl import load_workbook
from openpyxl.styles import Font, PatternFill, Alignment
from openpyxl.chart import BarChart, Reference

# Configuration
CONFIG = {
    'input_path': 'sales_data_raw.xlsx',
    'output_path': f'sales_report_{datetime.now().strftime("%Y%m%d")}.xlsx',
    'date_formats': ['%Y-%m-%d', '%d/%m/%Y', '%m-%d-%Y', '%d-%m-%Y'],
    'required_columns': ['order_id', 'order_date', 'customer_name', 
                         'product', 'quantity', 'price', 'status'],
    'valid_status': ['Completed', 'Cancelled', 'Pending']
}

# Column mapping for messy inputs
COLUMN_ALIASES = {
    'order_id': ['OrderID', 'Order ID', 'order id', 'ID', 'Order_No', 'order no'],
    'order_date': ['Date', 'Order Date', 'OrderDate', 'Purchase Date', 'date'],
    'customer_name': ['Customer', 'Cust Name', 'customer_name', 'Client', 'customer'],
    'product': ['Product', 'product_name', 'Item', 'product name'],
    'quantity': ['Qty', 'quantity', 'Quantity', 'qty', 'amount'],
    'price': ['Price', 'Unit Price', 'unit_price', 'cost', 'price'],
    'region': ['Region', 'region', 'Location', 'location', 'area'],
    'sales_rep': ['Sales Rep', 'sales_rep', 'Rep', 'Salesperson', 'rep'],
    'status': ['Status', 'status', 'Order Status', 'state']
}

print("✓ Configuration loaded")
```


***

### **Section 2: Extract \& Validate (20 minutes)**

```python
# 📥 STEP 1: EXTRACTION
def load_excel_smart(filepath):
    """Load Excel with smart sheet detection"""
    print(f"[EXTRACT] Loading file: {filepath}")
    
    # Read all sheets
    excel_file = pd.ExcelFile(filepath)
    print(f"[INFO] Found {len(excel_file.sheet_names)} sheets: {excel_file.sheet_names}")
    
    # Find the data sheet (largest non-empty sheet)
    max_rows = 0
    data_sheet = None
    
    for sheet in excel_file.sheet_names:
        df_temp = pd.read_excel(filepath, sheet_name=sheet)
        if len(df_temp) > max_rows:
            max_rows = len(df_temp)
            data_sheet = sheet
    
    df = pd.read_excel(filepath, sheet_name=data_sheet)
    print(f"[EXTRACT] ✓ Loaded {len(df)} rows from sheet '{data_sheet}'")
    return df

# 🔍 STEP 2: COLUMN MAPPING
def normalize_columns(df):
    """Map messy column names to standard names"""
    print("\n[VALIDATE] Normalizing column names...")
    
    # Clean column names (lowercase, strip spaces)
    df.columns = df.columns.str.strip().str.lower()
    
    # Map to standard names
    reverse_mapping = {}
    for standard, aliases in COLUMN_ALIASES.items():
        for alias in aliases:
            reverse_mapping[alias.lower()] = standard
    
    df.rename(columns=reverse_mapping, inplace=True)
    
    # Check required columns
    missing = set(CONFIG['required_columns']) - set(df.columns)
    if missing:
        print(f"[ERROR] Missing required columns: {missing}")
        print(f"[INFO] Available columns: {df.columns.tolist()}")
        raise ValueError(f"Missing columns: {missing}")
    
    print(f"[VALIDATE] ✓ Columns normalized: {df.columns.tolist()}")
    return df

# 📊 STEP 3: VALIDATION REPORT
def generate_validation_report(df):
    """Generate data quality metrics"""
    print("\n[VALIDATE] Generating data quality report...")
    
    report = {
        'total_rows': len(df),
        'total_columns': len(df.columns),
        'duplicate_rows': df.duplicated().sum(),
        'null_counts': df.isnull().sum().to_dict(),
        'null_percentage': (df.isnull().sum() / len(df) * 100).round(2).to_dict()
    }
    
    print(f"[VALIDATE] Total Rows: {report['total_rows']}")
    print(f"[VALIDATE] Duplicates: {report['duplicate_rows']}")
    print(f"[VALIDATE] Columns with nulls: {sum(1 for v in report['null_counts'].values() if v > 0)}")
    
    return report

# 🚀 EXECUTE EXTRACTION
df_raw = load_excel_smart(CONFIG['input_path'])
df = normalize_columns(df_raw)
validation_report = generate_validation_report(df)
```


***

### **Section 3: Clean \& Transform (30 minutes)**

```python
# 🧹 STEP 4: DATA CLEANING
def clean_data(df):
    """Apply all cleaning transformations"""
    print("\n[CLEAN] Starting data cleaning pipeline...")
    
    df_clean = df.copy()
    initial_rows = len(df_clean)
    
    # 1. Remove exact duplicates
    df_clean = df_clean.drop_duplicates()
    print(f"[CLEAN] Removed {initial_rows - len(df_clean)} duplicate rows")
    
    # 2. Remove rows where order_id is null
    df_clean = df_clean.dropna(subset=['order_id'])
    print(f"[CLEAN] Removed rows with null order_id")
    
    # 3. Standardize status values
    if 'status' in df_clean.columns:
        df_clean['status'] = df_clean['status'].str.strip().str.title()
        df_clean['status'] = df_clean['status'].replace({
            'Complete': 'Completed',
            'Cancel': 'Cancelled',
            'Canceled': 'Cancelled',
            'In Progress': 'Pending'
        })
    
    # 4. Fill missing customer names
    df_clean['customer_name'] = df_clean['customer_name'].fillna('Unknown Customer')
    
    # 5. Fix date formats
    df_clean['order_date'] = pd.to_datetime(
        df_clean['order_date'], 
        errors='coerce',
        infer_datetime_format=True
    )
    
    # Remove rows with invalid dates
    invalid_dates = df_clean['order_date'].isnull().sum()
    df_clean = df_clean.dropna(subset=['order_date'])
    print(f"[CLEAN] Removed {invalid_dates} rows with invalid dates")
    
    # 6. Clean numeric columns
    df_clean['quantity'] = pd.to_numeric(df_clean['quantity'], errors='coerce').fillna(0)
    df_clean['price'] = pd.to_numeric(df_clean['price'], errors='coerce').fillna(0)
    
    # 7. Remove negative values
    df_clean = df_clean[(df_clean['quantity'] >= 0) & (df_clean['price'] >= 0)]
    
    print(f"[CLEAN] ✓ Cleaned data: {len(df_clean)} rows retained ({len(df_clean)/initial_rows*100:.1f}%)")
    return df_clean

# 🔄 STEP 5: TRANSFORMATIONS
def add_calculated_columns(df):
    """Add derived columns"""
    print("\n[TRANSFORM] Adding calculated columns...")
    
    df_transform = df.copy()
    
    # 1. Total amount
    df_transform['total_amount'] = df_transform['quantity'] * df_transform['price']
    
    # 2. Date components
    df_transform['year'] = df_transform['order_date'].dt.year
    df_transform['month'] = df_transform['order_date'].dt.month
    df_transform['month_name'] = df_transform['order_date'].dt.strftime('%B')
    df_transform['day_of_week'] = df_transform['order_date'].dt.day_name()
    df_transform['quarter'] = df_transform['order_date'].dt.quarter
    
    # 3. Revenue category
    df_transform['revenue_category'] = pd.cut(
        df_transform['total_amount'],
        bins=[0, 100, 500, 1000, float('inf')],
        labels=['Low', 'Medium', 'High', 'Premium']
    )
    
    # 4. Days since order
    df_transform['days_since_order'] = (datetime.now() - df_transform['order_date']).dt.days
    
    print(f"[TRANSFORM] ✓ Added calculated columns")
    return df_transform

# 🚀 EXECUTE CLEANING
df_clean = clean_data(df)
df_final = add_calculated_columns(df_clean)

print(f"\n[INFO] Final dataset shape: {df_final.shape}")
print(f"[INFO] Columns: {df_final.columns.tolist()}")
```


***

### **Section 4: Business Logic \& KPIs (25 minutes)**

```python
# 📊 STEP 6: CALCULATE KPIs
def calculate_kpis(df):
    """Calculate key business metrics"""
    print("\n[ANALYSIS] Calculating KPIs...")
    
    # Filter completed orders for revenue metrics
    df_completed = df[df['status'] == 'Completed']
    
    kpis = {
        'total_orders': len(df),
        'completed_orders': len(df_completed),
        'cancelled_orders': len(df[df['status'] == 'Cancelled']),
        'pending_orders': len(df[df['status'] == 'Pending']),
        'total_revenue': df_completed['total_amount'].sum(),
        'avg_order_value': df_completed['total_amount'].mean(),
        'total_customers': df['customer_name'].nunique(),
        'total_products': df['product'].nunique(),
        'completion_rate': len(df_completed) / len(df) * 100,
        'cancellation_rate': len(df[df['status'] == 'Cancelled']) / len(df) * 100,
        'date_range_start': df['order_date'].min(),
        'date_range_end': df['order_date'].max()
    }
    
    print(f"[ANALYSIS] ✓ KPIs calculated")
    return kpis

# 📈 STEP 7: AGGREGATIONS
def create_aggregations(df):
    """Create summary tables"""
    print("\n[ANALYSIS] Creating aggregations...")
    
    df_completed = df[df['status'] == 'Completed']
    
    # 1. Sales by Month
    sales_by_month = df_completed.groupby(['year', 'month', 'month_name']).agg({
        'order_id': 'count',
        'total_amount': 'sum',
        'customer_name': 'nunique'
    }).rename(columns={
        'order_id': 'orders',
        'total_amount': 'revenue',
        'customer_name': 'unique_customers'
    }).reset_index()
    
    # 2. Top Products
    top_products = df_completed.groupby('product').agg({
        'order_id': 'count',
        'quantity': 'sum',
        'total_amount': 'sum'
    }).rename(columns={
        'order_id': 'orders',
        'quantity': 'units_sold',
        'total_amount': 'revenue'
    }).sort_values('revenue', ascending=False).head(10).reset_index()
    
    # 3. Top Customers
    top_customers = df_completed.groupby('customer_name').agg({
        'order_id': 'count',
        'total_amount': 'sum'
    }).rename(columns={
        'order_id': 'orders',
        'total_amount': 'lifetime_value'
    }).sort_values('lifetime_value', ascending=False).head(10).reset_index()
    
    # 4. Status Summary
    status_summary = df.groupby('status').agg({
        'order_id': 'count',
        'total_amount': 'sum'
    }).rename(columns={
        'order_id': 'count',
        'total_amount': 'total_value'
    }).reset_index()
    
    # 5. Revenue by Category
    revenue_by_category = df_completed.groupby('revenue_category').agg({
        'order_id': 'count',
        'total_amount': 'sum'
    }).rename(columns={
        'order_id': 'orders',
        'total_amount': 'revenue'
    }).reset_index()
    
    # Add region analysis if available
    aggregations = {
        'sales_by_month': sales_by_month,
        'top_products': top_products,
        'top_customers': top_customers,
        'status_summary': status_summary,
        'revenue_by_category': revenue_by_category
    }
    
    if 'region' in df.columns:
        sales_by_region = df_completed.groupby('region').agg({
            'order_id': 'count',
            'total_amount': 'sum'
        }).rename(columns={
            'order_id': 'orders',
            'total_amount': 'revenue'
        }).sort_values('revenue', ascending=False).reset_index()
        aggregations['sales_by_region'] = sales_by_region
    
    print(f"[ANALYSIS] ✓ Created {len(aggregations)} aggregation tables")
    return aggregations

# 🚀 EXECUTE ANALYSIS
kpis = calculate_kpis(df_final)
aggregations = create_aggregations(df_final)

# Display KPIs
print("\n" + "="*50)
print("KEY PERFORMANCE INDICATORS")
print("="*50)
for key, value in kpis.items():
    if isinstance(value, (int, float)):
        if 'rate' in key or 'percentage' in key:
            print(f"{key.replace('_', ' ').title()}: {value:.2f}%")
        elif 'revenue' in key or 'value' in key:
            print(f"{key.replace('_', ' ').title()}: ${value:,.2f}")
        else:
            print(f"{key.replace('_', ' ').title()}: {value:,}")
    else:
        print(f"{key.replace('_', ' ').title()}: {value}")
print("="*50)
```


***

### **Section 5: Generate Reports (30 minutes)**

```python
# 📝 STEP 8: CREATE SUMMARY SHEET
def create_summary_sheet(kpis):
    """Create executive summary DataFrame"""
    summary_data = {
        'Metric': [],
        'Value': []
    }
    
    for key, value in kpis.items():
        metric_name = key.replace('_', ' ').title()
        
        if isinstance(value, (int, float)):
            if 'rate' in key or 'percentage' in key:
                formatted_value = f"{value:.2f}%"
            elif 'revenue' in key or 'value' in key:
                formatted_value = f"${value:,.2f}"
            else:
                formatted_value = f"{value:,}"
        else:
            formatted_value = str(value)
        
        summary_data['Metric'].append(metric_name)
        summary_data['Value'].append(formatted_value)
    
    return pd.DataFrame(summary_data)

# 📊 STEP 9: EXPORT TO EXCEL
def export_to_excel(df_final, kpis, aggregations, validation_report, output_path):
    """Export all data to formatted Excel file"""
    print(f"\n[EXPORT] Creating Excel report: {output_path}")
    
    # Create Excel writer
    with pd.ExcelWriter(output_path, engine='openpyxl') as writer:
        
        # Sheet 1: Executive Summary
        summary_df = create_summary_sheet(kpis)
        summary_df.to_excel(writer, sheet_name='Executive Summary', index=False)
        print("[EXPORT] ✓ Sheet 1: Executive Summary")
        
        # Sheet 2: Sales by Month
        aggregations['sales_by_month'].to_excel(writer, sheet_name='Sales by Month', index=False)
        print("[EXPORT] ✓ Sheet 2: Sales by Month")
        
        # Sheet 3: Top Products
        aggregations['top_products'].to_excel(writer, sheet_name='Top Products', index=False)
        print("[EXPORT] ✓ Sheet 3: Top Products")
        
        # Sheet 4: Top Customers
        aggregations['top_customers'].to_excel(writer, sheet_name='Top Customers', index=False)
        print("[EXPORT] ✓ Sheet 4: Top Customers")
        
        # Sheet 5: Status Summary
        aggregations['status_summary'].to_excel(writer, sheet_name='Status Summary', index=False)
        print("[EXPORT] ✓ Sheet 5: Status Summary")
        
        # Sheet 6: Revenue Categories
        aggregations['revenue_by_category'].to_excel(writer, sheet_name='Revenue Categories', index=False)
        print("[EXPORT] ✓ Sheet 6: Revenue Categories")
        
        # Sheet 7: Region Analysis (if available)
        if 'sales_by_region' in aggregations:
            aggregations['sales_by_region'].to_excel(writer, sheet_name='Sales by Region', index=False)
            print("[EXPORT] ✓ Sheet 7: Sales by Region")
        
        # Sheet 8: Data Quality Report
        quality_df = pd.DataFrame([
            {'Metric': 'Total Rows (Raw)', 'Value': validation_report['total_rows']},
            {'Metric': 'Total Rows (Clean)', 'Value': len(df_final)},
            {'Metric': 'Duplicates Removed', 'Value': validation_report['duplicate_rows']},
            {'Metric': 'Data Quality Score', 'Value': f"{(len(df_final)/validation_report['total_rows']*100):.1f}%"}
        ])
        quality_df.to_excel(writer, sheet_name='Data Quality', index=False)
        print("[EXPORT] ✓ Sheet 8: Data Quality Report")
        
        # Sheet 9: Clean Data (first 1000 rows)
        df_final.head(1000).to_excel(writer, sheet_name='Clean Data Sample', index=False)
        print("[EXPORT] ✓ Sheet 9: Clean Data Sample")
    
    print(f"\n[EXPORT] ✓✓ Excel report generated: {output_path}")
    return output_path

# 🎨 STEP 10: FORMAT EXCEL (Optional - adds 5-10 min)
def format_excel(filepath):
    """Apply formatting to Excel file"""
    print("\n[FORMAT] Applying Excel formatting...")
    
    wb = load_workbook(filepath)
    
    # Format each sheet
    for sheet_name in wb.sheetnames:
        ws = wb[sheet_name]
        
        # Header formatting
        header_fill = PatternFill(start_color='366092', end_color='366092', fill_type='solid')
        header_font = Font(bold=True, color='FFFFFF')
        
        for cell in ws[1]:
            cell.fill = header_fill
            cell.font = header_font
            cell.alignment = Alignment(horizontal='center', vertical='center')
        
        # Auto-adjust column widths
        for column in ws.columns:
            max_length = 0
            column_letter = column[0].column_letter
            for cell in column:
                try:
                    if len(str(cell.value)) > max_length:
                        max_length = len(str(cell.value))
                except:
                    pass
            adjusted_width = min(max_length + 2, 50)
            ws.column_dimensions[column_letter].width = adjusted_width
    
    wb.save(filepath)
    print("[FORMAT] ✓ Excel formatting applied")

# 🚀 EXECUTE EXPORT
output_file = export_to_excel(df_final, kpis, aggregations, validation_report, CONFIG['output_path'])

# Optional: Apply formatting (comment out if short on time)
format_excel(output_file)
```


***

### **Section 6: Testing \& Validation (10 minutes)**

```python
# ✅ STEP 11: FINAL VALIDATION
print("\n" + "="*60)
print("FINAL PIPELINE VALIDATION")
print("="*60)

# 1. Check output file exists
import os
if os.path.exists(output_file):
    file_size = os.path.getsize(output_file) / 1024  # KB
    print(f"✓ Output file created: {output_file}")
    print(f"✓ File size: {file_size:.2f} KB")
else:
    print("✗ Output file NOT created!")

# 2. Verify data integrity
print(f"\n✓ Input rows: {validation_report['total_rows']:,}")
print(f"✓ Output rows: {len(df_final):,}")
print(f"✓ Data retention rate: {(len(df_final)/validation_report['total_rows']*100):.1f}%")

# 3. Business metrics check
print(f"\n✓ Total Revenue: ${kpis['total_revenue']:,.2f}")
print(f"✓ Completion Rate: {kpis['completion_rate']:.1f}%")
print(f"✓ Unique Customers: {kpis['total_customers']:,}")
print(f"✓ Unique Products: {kpis['total_products']:,}")

# 4. Show sample of clean data
print("\n" + "="*60)
print("SAMPLE CLEAN DATA (First 5 rows)")
print("="*60)
display(df_final[['order_id', 'order_date', 'customer_name', 'product', 
                   'quantity', 'price', 'total_amount', 'status']].head())

# 5. Show top products
print("\n" + "="*60)
print("TOP 5 PRODUCTS BY REVENUE")
print("="*60)
display(aggregations['top_products'].head())

print("\n" + "="*60)
print("✓✓ PIPELINE COMPLETED SUCCESSFULLY!")
print("="*60)
print(f"\n📊 Report Location: {output_file}")
print(f"⏱️  Processing Time: ~2 minutes")
print(f"📈 Ready for Business Review!")
```


***

## 📦 **Bonus: Generate Sample Messy Data** (Optional - for testing)

```python
# 🎲 CREATE SAMPLE MESSY DATA (Run this first to create test file)
def create_messy_sample_data():
    """Generate sample messy Excel file for testing"""
    np.random.seed(42)
    
    n_rows = 500
    
    # Mix of good and messy data
    data = {
        'Order ID': range(1001, 1001 + n_rows),  # Inconsistent column name
        'order date': [  # Inconsistent dates
            (datetime.now() - timedelta(days=np.random.randint(0, 365))).strftime(
                np.random.choice(['%Y-%m-%d', '%d/%m/%Y', '%m-%d-%Y'])
            ) for _ in range(n_rows)
        ],
        'Cust Name': np.random.choice(  # Missing values
            ['Acme Corp', 'TechStart Inc', 'Global Solutions', None, 'Elite Systems', 'Unknown'],
            n_rows, p=[0.3, 0.25, 0.2, 0.1, 0.1, 0.05]
        ),
        'product_name': np.random.choice(
            ['Laptop', 'Mouse', 'Keyboard', 'Monitor', 'Headphones', 'Webcam'],
            n_rows
        ),
        'Qty': np.random.randint(1, 20, n_rows),
        'Unit Price': np.round(np.random.uniform(10, 500, n_rows), 2),
        'Region': np.random.choice(['North', 'South', 'East', 'West'], n_rows),
        'Sales Rep': np.random.choice(['John', 'Sarah', 'Mike', 'Lisa'], n_rows),
        'Order Status': np.random.choice(
            ['Completed', 'Cancelled', 'Pending', 'Complete'],  # Inconsistent values
            n_rows, p=[0.7, 0.15, 0.1, 0.05]
        )
    }
    
    df_messy = pd.DataFrame(data)
    
    # Add duplicates
    duplicates = df_messy.sample(20)
    df_messy = pd.concat([df_messy, duplicates]).reset_index(drop=True)
    
    # Add some null prices
    null_indices = np.random.choice(df_messy.index, 10, replace=False)
    df_messy.loc[null_indices, 'Unit Price'] = None
    
    # Create Excel with multiple sheets (to test smart loading)
    with pd.ExcelWriter('sales_data_raw.xlsx', engine='openpyxl') as writer:
        pd.DataFrame({'info': ['This is metadata']}).to_excel(writer, sheet_name='Info', index=False)
        df_messy.to_excel(writer, sheet_name='Sales Data', index=False)
        pd.DataFrame({'notes': ['Empty sheet']}).to_excel(writer, sheet_name='Notes', index=False)
    
    print("✓ Sample messy data created: sales_data_raw.xlsx")
    print(f"  - {len(df_messy)} rows (including duplicates)")
    print(f"  - 3 sheets (Sales Data is the largest)")
    print(f"  - Inconsistent column names, dates, nulls")
    return df_messy

# Uncomment to generate test data
# create_messy_sample_data()
```


***

## ⏱️ **Time Breakdown**

| Section | Time | Deliverable |
| :-- | :-- | :-- |
| Setup \& Config | 5 min | Configuration complete |
| Extract \& Validate | 20 |  |

