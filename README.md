**Voxta: Real-Time Chat Application**
Voxta is a real-time chat application built with Django (backend) and React (frontend). It allows users to connect by sending and accepting interests, enabling private messaging only between mutually connected users. The app uses WebSocket for real-time communication, powered by Django Channels and Redis, with a robust authentication system using JWT.
Features

User Authentication: Register, login, and logout with JWT-based authentication.
Interest-Based Connections: Send, accept, or reject connection requests to build a network.
Real-Time Chat: Private messaging with WebSocket, including typing indicators.
Searchable User List: Browse and search users by username or email.
Responsive UI: Modern, user-friendly interface with Tailwind CSS and Lucide icons.
Scalable Backend: Django with Django Channels for real-time features.
Planned Encryption: HTTPS/WSS for transport-layer security (production); end-to-end encryption optional.

Tech Stack

Frontend: React, Redux Toolkit, Axios, Tailwind CSS, Lucide Icons, Vite
Backend: Django, Django REST Framework, Django Channels, SimpleJWT
Database: SQLite3 (development), PostgreSQL (production)
Real-Time: WebSocket (Django Channels), Redis
Deployment: Vercel (frontend), AWS EC2/RDS (backend)
Others: Nginx (reverse proxy), Certbot (SSL), Supervisor (process management)

Project Structure
voxta/
├── voxta_backend/              # Django backend
│   ├── user_app/               # Custom user app (models, views, consumers)
│   ├── voxta_backend/          # Django settings, ASGI, URLs
│   ├── staticfiles/            # Collected static files
│   ├── media/                  # User-uploaded media
│   └── manage.py
├── frontend/                   # React frontend
│   ├── src/
│   │   ├── components/         # React components (Chat, BrowseUsers)
│   │   ├── store/              # Redux store and slices
│   │   ├── API/                # Axios API client
│   │   └── utils/              # Utility functions (e.g., encryption)
│   ├── public/
│   └── package.json
├── .env                        # Environment variables
└── README.md

Prerequisites

Node.js: v18 or higher
Python: v3.8 or higher
Redis: v6 or higher
PostgreSQL: v13 or higher (production)
AWS Account: For backend deployment
Vercel Account: For frontend deployment
Git: For cloning the repository

Installation
Development Setup (SQLite3)

Clone the Repository:
git clone https://github.com/<your-username>/voxta.git
cd voxta


Backend Setup:
cd voxta_backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt


Create .env in voxta_backend/:SECRET_KEY=your-secret-key
DEBUG=True


Run migrations:python manage.py makemigrations
python manage.py migrate


Start Redis:redis-server


Run Daphne:daphne -b 0.0.0.0 -p 8000 voxta_backend.asgi:application




Frontend Setup:
cd ../frontend
npm install
npm run dev


Ensure axiosInstance.js points to http://localhost:8000/api/.
Update Chat.jsx WebSocket URL to ws://localhost:8000/ws/chat/.


Test the App:

Open http://localhost:5173 in your browser.
Register, send interests, and chat.



Production Setup (PostgreSQL on AWS)

Backend Deployment (AWS EC2/RDS):

Set Up RDS PostgreSQL:
Create a PostgreSQL instance in AWS RDS (e.g., db.t3.micro).
Note the endpoint, username, password, and database name.


Set Up EC2:ssh -i your-key.pem ec2-user@<ec2-public-ip>
sudo yum update -y
sudo yum install python3 python3-pip redis nginx git -y
git clone https://github.com/<your-username>/voxta.git
cd voxta/voxta_backend
python3 -m venv venv
source venv/bin/activate
pip install django djangorestframework django-channels daphne psycopg2-binary django-cors-headers redis


Configure .env:SECRET_KEY=your-secret-key
DEBUG=False
DATABASE_URL=postgresql://username:password@rds-endpoint:5432/dbname


Run migrations:python manage.py migrate
python manage.py collectstatic


Start Redis:sudo systemctl start redis
sudo systemctl enable redis


Set up Supervisor:sudo yum install supervisor -y
sudo nano /etc/supervisord.d/voxta.ini

Add:[program:voxta]
command=/home/ec2-user/voxta/voxta_backend/venv/bin/daphne -b 0.0.0.0 -p 8000 voxta_backend.asgi:application
directory=/home/ec2-user/voxta/voxta_backend
user=ec2-user
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/var/log/voxta.log

Start:sudo systemctl start supervisord
sudo systemctl enable supervisord


Configure Nginx:sudo nano /etc/nginx/nginx.conf

Add:server {
    listen 80;
    server_name <ec2-public-ip> <your-domain>;
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    location /ws/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
    location /static/ {
        alias /home/ec2-user/voxta/voxta_backend/staticfiles/;
    }
    location /media/ {
        alias /home/ec2-user/voxta/voxta_backend/media/;
    }
}

Start:sudo systemctl start nginx
sudo systemctl enable nginx


Enable HTTPS:sudo yum install certbot python3-certbot-nginx -y
sudo certbot --nginx -d <your-domain>




Migrate Data from SQLite3:
python manage.py dumpdata --exclude auth.permission --exclude contenttypes > data.json
python manage.py loaddata data.json


Frontend Deployment (Vercel):

Push frontend to GitHub:cd frontend
git add .
git commit -m "Deploy to Vercel"
git push origin main


In Vercel:
Import the repository.
Set Framework: React.
Build Command: npm run build.
Output Directory: dist.
Deploy.


Update axiosInstance.js:baseURL: 'https://<your-domain>/api/'


Update Chat.jsx:const wsUrl = `wss://<your-domain>/ws/chat/`;




Update CORS:

In voxta_backend/settings.py:CORS_ALLOWED_ORIGINS = [
    "https://<your-vercel-app>.vercel.app",
    "https://<your-domain>",
]




Test Production:

Visit https://<your-vercel-app>.vercel.app.
Test registration, interest sending, and real-time chat.
Check AWS logs:tail -f /var/log/voxta.log
sudo tail -f /var/log/nginx/error.log





Usage

Register/Login: Create an account or sign in.
Browse Users: Search users and send connection requests.
Manage Interests: Accept or reject incoming requests.
Chat: Message mutually connected users in real-time.

Security

Authentication: JWT tokens stored in HTTP-only cookies.
Transport Encryption: HTTPS/WSS (production).
End-to-End Encryption: Not implemented (optional; see utils/encryption.js).
Best Practices:
Restrict RDS access to EC2 via VPC.
Rotate SECRET_KEY and database credentials.
Use rate limiting for WebSocket messages.



Contributing

Fork the repository.
Create a feature branch: git checkout -b feature-name.
Commit changes: git commit -m 'Add feature'.
Push: git push origin feature-name.
Open a pull request.

License
MIT License. See LICENSE for details.
Contact
For issues or questions, open a GitHub issue or contact <your-email>.
