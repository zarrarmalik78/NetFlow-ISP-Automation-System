release: python manage.py migrate
web: gunicorn netflow_backend.wsgi:application --bind 0.0.0.0:$PORT
