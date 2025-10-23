# Step 1: Clone the repository
- Clone from GitHub git clone https://github.com/YOUR_USERNAME/arise-system.git  
- Navigate to project directory cd ARISE  

# Step 2: Set Up Python Virtual Environment
- Create virtual environment   
python -m venv venv  
- Activate virtual environment   
venv\Scripts\activate  

# Step 3: Install Python Dependencies
- Upgrade pip   
pip install --upgrade pip   
- Install all required packages   
pip install -r requirements.txt  

# Step 4: Install PostgreSQL + PostGIS
Option A: PostgreSQL Installer (Recommended)  
Download PostgreSQL:  
- Go to: https://www.postgresql.org/download/windows/
- Download PostgreSQL 13 or higher installer (e.g., postgresql-13.x-windows-x64.exe)
- Run Installer:  
- Double-click the installer
- Click Next through the welcome screen
- Installation Directory: Keep default (C:\Program Files\PostgreSQL\13)
- Select Components: Check ALL boxes (especially Stack Builder)
- Data Directory: Keep default
- Password: Set a strong password (remember this!)
- Port: Keep default 5432
- Locale: Keep default
- Click Next and Finish
Install PostGIS via Stack Builder:  
- Stack Builder should open automatically (if not, find it in Start Menu)
- Select your PostgreSQL installation
- Expand Spatial Extensions
- Check PostGIS 3.x Bundle for PostgreSQL
- Click Next and install
- Accept defaults in PostGIS installer
- Complete installation
Verify Installation:
Open Command Prompt
   psql --version
Should show:  
psql (PostgreSQL) 13.x  

# Step 5: Install GDAL
GDAL is CRITICAL for processing geospatial data. Follow these steps carefully.  
Method 1: OSGeo4W (Recommended)  
Step-by-step:  
- Download OSGeo4W Installer:
- Go to: https://trac.osgeo.org/osgeo4w/
- Download OSGeo4W Network Installer (64-bit): osgeo4w-setup.exe
- Save to your Downloads folder
Run Installer as Administrator:  
- Right-click osgeo4w-setup.exe → Run as administrator
Installation Wizard:  
- Select Installation Type: Choose Express Install
- Select Packages: Check the following:  
- ✅ GDAL (Geospatial Data Abstraction Library)
- ✅ QGIS (Optional but recommended for viewing data)
- ✅ GRASS GIS (Optional)
- Installation Directory: Keep default C:\OSGeo4W64 (or C:\OSGeo4W for 32-bit)  
- Click Next and wait for download/installation (5-10 minutes)

Verify GDAL Installation:
- Open OSGeo4W Shell (search in Start Menu)
gdalinfo --version  
Should show: GDAL 3.x.x, released 202x/xx/xx

**Add GDAL to System PATH:**  
**Option A: Automatic (Easier)**  
   - Open **OSGeo4W Shell** (search in Start Menu)
   - Type: `py3_env` (this sets up Python environment)
   - Type: `setx PATH "%PATH%;C:\OSGeo4W64\bin"` (for 64-bit)
   - **Restart your computer**  

**Option B: Manual (More Control)**  
a. **Open Environment Variables:**  
- Press `Win + X` → System
- Click **Advanced system settings**
- Click **Environment Variables** button

b. **Edit System PATH:**  
- Under **System variables**, find **Path**
- Click **Edit**
- Click **New** and add these paths **one by one**:
- C:\OSGeo4W64\bin
- C:\OSGeo4W64\apps\Python39\Scripts
- C:\OSGeo4W64\apps\gdal\bin
- Click **OK** on all windows

c. Add GDAL Environment Variables:  
- Still in Environment Variables window
- Under System variables, click New
- Add these variables:  
| Variable Name | Variable Value |  
|--------------|----------------|  
| `GDAL_DATA` | `C:\OSGeo4W64\share\gdal` |  
| `PROJ_LIB` | `C:\OSGeo4W64\share\proj` |  
| `GDAL_DRIVER_PATH` | `C:\OSGeo4W64\bin\gdalplugins` |

d. Verify PATH Configuration:  
- Close ALL Command Prompts/PowerShell windows
- Open NEW Command Prompt
gdalinfo --version  
- Should work now!    
- Test PROJ
projinfo  
- Should show PROJ version info

Fix Common Issues:  
Issue: gdalinfo command not found  
Solution: Restart computer after adding to PATH  
Verify PATH contains: C:\OSGeo4W64\bin  
Issue: DLL errors  
Add to PATH:  
- C:\OSGeo4W64\bin
- C:\OSGeo4W64\apps\Python39\DLLs

# Step 6: Configure PostgreSQL Database
A. Start PostgreSQL Service  
  Windows:  
    •	Search "Services" → Find "postgresql-x64-13" → Start  
B. Create Database  

- Access PostgreSQL shell  
psql -U postgres

- Inside psql shell:
CREATE DATABASE arise;  
CREATE USER arise_user WITH PASSWORD 'your_password_here';  
ALTER ROLE arise_user SET client_encoding TO 'utf8';  
ALTER ROLE arise_user SET default_transaction_isolation TO 'read committed';  
ALTER ROLE arise_user SET timezone TO 'Asia/Manila';  
GRANT ALL PRIVILEGES ON DATABASE arise TO arise_user;  

- Enable PostGIS extension
\c arise  
CREATE EXTENSION postgis;  

- Exit psql  
\q  

# Step 7: Restore Database from Backup  
Database Backup Link: https://drive.google.com/drive/u/5/folders/1L8TE_teKz3ENe0woNetcBmOiwCRsglGd  
Option A: Using pgAdmin 
Copy Backup File:  
- Locate arise_database_backup.backup (or .sql file) from the repository
- Note the file location
  
Restore via pgAdmin:  
- Open pgAdmin 4
- Expand Servers → PostgreSQL 13
- Right-click arise database → Restore
- Format: Select based on your file:
If .backup → Custom or tar  
If .sql → Plain  

- Filename: Browse and select arise_database_backup.backup
- Role name: arise_user
- Click Restore
- Wait for completion (may take 2-5 minutes)

Verify Restoration:  
- Right-click arise database → Query Tool
- Run:  
     SELECT COUNT(*) FROM hazard_maps_barangayboundarynew;  
     -- Should return number of barangays (e.g., 557)  
     
     SELECT COUNT(*) FROM hazard_maps_floodsusceptibility;  
     -- Should return number of flood records  

# Step 8: Configure Django Settings
A. Update Database Settings  
Open hazard_system/settings.py and update:  
DATABASES = {  
    'default': {  
        'ENGINE': 'django.contrib.gis.db.backends.postgis',  
        'NAME': 'arise',  
        'USER': 'arise_user',  # Change this  
        'PASSWORD': 'your_password_here',  # Change this  
        'HOST': 'localhost',  
        'PORT': '5432',  
    }  
}  

B. Configure GDAL/GEOS Paths (Windows Only)  
In hazard_system/settings.py, update the OSGeo4W path if different:  
if os.name == 'nt':  # Windows  
    OSGEO4W = r"C:\OSGeo4W64"  # or C:\OSGeo4W  
    
    GDAL_LIBRARY_PATH = os.path.join(OSGEO4W, 'bin', 'gdal311.dll')  
    GEOS_LIBRARY_PATH = os.path.join(OSGEO4W, 'bin', 'geos_c.dll')  
    
    os.environ['GDAL_DATA'] = os.path.join(OSGEO4W, 'share', 'gdal')  
    os.environ['PROJ_LIB'] = os.path.join(OSGEO4W, 'share', 'proj')  

C. Update Secret Key  
SECRET_KEY = 'django-insecure-CHANGE-THIS-TO-A-RANDOM-STRING'  
Generate a new secret key:  
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"  

# Step 9: Run Database Migrations
- Create migrations
python manage.py makemigrations
- Apply migrations
python manage.py migrate  
- Create cache table
python manage.py createcachetable

# Step 10: Create Admin User
python manage.py createsuperuser
- Follow prompts:
- Username: admin
- Email: your_email@example.com
- Password: (create a strong password)

# Step 11: Collect Static Files (Production Only)
python manage.py collectstatic --noinput  

# Step 12: Run the Development Server
python manage.py runserver  
- Open your browser and go to: http://127.0.0.1:8000/




# Troubleshooting (Windows)
Error: GDAL library not found  
- Verify OSGeo4W installation path
- Update OSGEO4W path in settings.py
- Restart command prompt

Error: Could not connect to PostgreSQL  
- Check if PostgreSQL is running
- Windows: Services → postgresql
- Verify credentials in settings.py
- Check PostgreSQL port (default: 5432)

Error: No module named 'django.contrib.gis'  
- PostGIS not installed
- Follow Step 4 again to install PostGIS
- Verify extension  
psql -U postgres -d arise -c "SELECT PostGIS_Version();"

Error: Overpass API timeout  
- Increase timeout in overpass_client.py
- or wait a few minutes and try again
- Overpass API has rate limits

Error: Cache table not found  
python manage.py createcachetable  
<img width="477" height="557" alt="image" src="https://github.com/user-attachments/assets/0a9348c6-84cd-4228-b251-92916e3b3a8c" />










