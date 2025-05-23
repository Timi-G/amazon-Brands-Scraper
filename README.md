# Amazon Brands Scraper
## Setup and local execution

Ensure virtual environment (venv) is activated and install dependencies by the following:
- Navigate to Django root project directory in Windows Powershell.
- If venv is not created, follow instructions to do so [here](https://docs.python.org/3/library/venv.html).
- Activate venv with `.venv/Scripts/activate`.
- Install dependencies
  - From requirements.txt use:
    - `pip install -r requirements.txt`
  - Alternatively, install individual packages:
    - Install Django with `pip install Django`.
    - Install celery with `pip install celery`.
    - Install requests with `pip install requests`.
    - Install Django with `pip install beautifulsoup4`.
    - Install results with `pip install django-celery-results`.
    - Install beat with `pip install django-celery-beat`.
    - Install sqlalchemy with `pip install sqlalchemy`.
    - Install broker with `pip install redis`.

To view the products of a brand through a minimalistic yet beautiful frontend, set up your system as follows:
+ Ensure Node.js is installed — the version used for this project is v17.3.1. If it is not installed, download the installer [here](https://nodejs.org/en/download).
+ Install TailwindCSS by running `npm install -D tailwindcss`.
+ Initialize TailwindCSS by running `npx tailwindcss init`.
+ Install DasiyUI by running `npm i -D daisyui`.

**After all installations, apply necessary migrations for the Django project with `python manage.py migrate`.**

## Scheduling and managing periodic tasks
- Make sure to create a new superuser account with `python manage.py createsuperuser`.
- To go to the admin panel for your Django website:
  - Run `python manage.py runserver` in terminal.
  - Enter in the link `{localhost}/admin` into your browser (note that your localhost will display in terminal e.g. http://127.0.0.1:8000/admin).
  - Create new brands in 'Brands' like Nike, Microsoft, et cetera.
- To utilize scheduling and periodic tasking, open a new terminal:
  - Windows
    - First, start your broker. The redis application — which is the broker used for this project — has already been made available; simply run it this way:
      - Navigate to the root directory in your terminal.
      - Run the command `./Redis-x64-3.0.504/redis-server`.
  - Linux
    - Kindly follow the steps in the official [redis website](https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/install-redis-on-linux/)
    - After successful installation, simply type `redis-server` in your terminal and press 'enter' to start the server.
    - If redis still doesn't start after the above steps, maybe try checking your Linux distribution or try using Docker.
- To start the celery worker:
  - Open another terminal or Windows PowerShell with venv activated (see how to activate venv in the previous section).
  - Windows
    - On Windows, due to multiprocessing limits, you have to run celery as solo with the command `celery -A amazonProductListing worker -l info -P solo` from the root directory `amazon-Brands-Scraper`
  - Linux
    - Ensure you are in the root directory `amazon-Brands-Scraper` and start celery with the command `celery -A amazonProductListing worker -l info`.
- Now, you need to start celery beat which sends due tasks to the worker.
  - To start celery beats
    - Open Django Admin Panel (/admin).
    - Navigate to "Periodic Tasks".
    - Click "Add Periodic Task":
    - You can name Periodic Task as you wish, e.g Scrape Amazon Products Every 6 Hours
    - Task: "amazonbrands.tasks.scrape_amazon_products_for_all_brands_db"
    - Schedule Type: Interval
    - Every: 6 (or your preferred figure)
    - Period: Hours (or your preferred period)
    - You can include 'start time'
    - All other arguments can be left empty
    - Save
    - Check terminal celery worker was started to ensure celery task is received (a log message should show that scrape has started)
    - If task hasn't started, select your saved periodic task and choose option "Run selected tasks" from "Actions" dropdown. Tap Go
    - Scraping for created brands should have started. Confirm in terminal celery worker was started 
  - Alternatively, start celery beat in terminal:
    - Open another terminal or Windows PowerShell ensuring venv activated
    - From the amazonProductListing root directory, run `celery -A amazonProductListing beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler` in the terminal

  **Check code execution and frontend viewership section in this documentation for how to view the Django website**

    *Note that you can manually scrap products of a brand using django admin (see note at the end of this documentation)*

## Web scraping implementation
- Firstly, the scraper visits `amazon.com` and gets the URL of products of the specific brands defined in the admin panel, taking pagination into consideration.
  - To reduce the risk of being blocked by Amazon's website, several usable User-Agents have been sourced and saved for use in the scraping process.
- Information such as name, asin, and page are retrieved from each product URL.
- To get high-quality image URLs for each product, the scraper accesses each product page for products of defined brand.
- The information of the saved product is saved to the database.

## Code execution and frontend viewership
To run the Django website:
- Run command `npm run dev` in a seperate terminal with venv activated to utilize `DaisyUI`.
- Run `python manage.py runserver` in another terminal with venv activated to start the development server if it isn't running in a previously opened terminal.
- Navigate to the link of the localhost on your browser (again, note that your localhost will display in terminal);
  - A sample localhost link (which will direct to the homepage) looks like this: `http://127.0.0.1/8000`.
- `{localhost}/products` will direct you to the products page.
- The search bar can be used to search for the products of brands already saved in the database.

## Any assumptions or design decisions
- The code was written with Python 3.12 and should run successfully on any Python 3.1x version.
- The Django website was developed with Django 5.1.2.
- It is assumed that this Django project will be run in a Windows environment. However, for Linux, most of the steps are similar, except changes to some commands like the use of `sudo` and installation of redis-server.
  - The celery documentation contains helpful instructions to run celery on Linux environments.
- A large number of products do not have readily available sku; hence scraping for this info was jettisoned.

### Admin panel
You can use the credentials below to create an easy-to-remember superuser.
- Username: admin
- Email: admin@mail.com
- Password: admin

**Note:  For manual scrape operations, use the actions dropdown feature at the top of the 'Brands' page in the admin panel to scrape products of specific brands. Simply select brands you want to scrape for and choose action 'Scrape products for selected brands'**
