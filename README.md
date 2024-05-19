# job_recommendation_website
Creating a user-friendly AI-driven website to recommend learning resources and help with resume building based on job descriptions is an ambitious and exciting project

Sure! I'll provide you with the necessary source code files and instructions to create your AI-driven job recommendation and resume building website from scratch. Here's a step-by-step guide with the required code snippets for each part:

### Step 1: Setting Up Your Environment

1. **Create and Configure the GCP VM**:
   - Follow the instructions to create a VM with Ubuntu OS on GCP.
   - SSH into your VM.

2. **Install Necessary Tools**:
   ```sh
   sudo apt update
   sudo apt upgrade
   sudo apt install python3 python3-pip nginx git nodejs npm
   ```

### Step 2: Setting Up the Backend

1. **Set Up Flask Backend**:

   - Create a directory for your backend:
     ```sh
     mkdir job_recommendation_backend
     cd job_recommendation_backend
     ```

   - Create and activate a virtual environment:
     ```sh
     python3 -m venv venv
     source venv/bin/activate
     ```

   - Install Flask and other dependencies:
     ```sh
     pip install Flask transformers torch
     ```

   - Create a `app.py` file with the following content:
     ```python
     from flask import Flask, request, jsonify
     from transformers import pipeline

     app = Flask(__name__)

     nlp = pipeline('feature-extraction', model='bert-base-uncased')

     def analyze_job_description(job_description):
         features = nlp(job_description)
         # Further processing to determine required skills and preparations
         # This is a placeholder. You would add logic to analyze features.
         return ["Skill1", "Skill2", "Skill3"]

     @app.route('/analyze', methods=['POST'])
     def analyze():
         job_description = request.json['job_description']
         features = analyze_job_description(job_description)
         return jsonify({"recommended_skills": features})

     if __name__ == '__main__':
         app.run(host='0.0.0.0', port=5000)
     ```

   - Run your Flask app:
     ```sh
     python app.py
     ```

### Step 3: Setting Up the Frontend

1. **Set Up React Frontend**:

   - Open a new terminal or SSH session into your VM.
   - Create a directory for your frontend:
     ```sh
     mkdir job_recommendation_frontend
     cd job_recommendation_frontend
     ```

   - Create a new React project:
     ```sh
     npx create-react-app .
     ```

   - Navigate to the `src` directory and create a file `JobForm.js` with the following content:
     ```javascript
     import React, { useState } from 'react';

     function JobForm() {
         const [jobDescription, setJobDescription] = useState('');
         const [recommendations, setRecommendations] = useState(null);

         const handleSubmit = async (e) => {
             e.preventDefault();
             const response = await fetch('/api/analyze', {
                 method: 'POST',
                 headers: {
                     'Content-Type': 'application/json'
                 },
                 body: JSON.stringify({ job_description: jobDescription })
             });
             const data = await response.json();
             setRecommendations(data.recommended_skills);
         };

         return (
             <div>
                 <form onSubmit={handleSubmit}>
                     <textarea 
                         value={jobDescription} 
                         onChange={(e) => setJobDescription(e.target.value)} 
                         placeholder="Paste job description here" 
                     />
                     <button type="submit">Analyze</button>
                 </form>
                 {recommendations && (
                     <div>
                         <h2>Recommended Skills</h2>
                         <ul>
                             {recommendations.map((skill, index) => (
                                 <li key={index}>{skill}</li>
                             ))}
                         </ul>
                     </div>
                 )}
             </div>
         );
     }

     export default JobForm;
     ```

   - Modify `src/App.js` to include the `JobForm` component:
     ```javascript
     import React from 'react';
     import JobForm from './JobForm';
     import './App.css';

     function App() {
         return (
             <div className="App">
                 <header className="App-header">
                     <h1>Job Recommendation and Resume Builder</h1>
                 </header>
                 <JobForm />
             </div>
         );
     }

     export default App;
     ```

   - Build the React app:
     ```sh
     npm run build
     ```

### Step 4: Deploying the Application

1. **Configure Nginx to Serve Both Backend and Frontend**:

   - Edit the Nginx configuration file:
     ```sh
     sudo nano /etc/nginx/sites-available/default
     ```

   - Update the configuration to proxy requests to the Flask backend and serve the React frontend:
     ```nginx
     server {
         listen 80;

         server_name your_domain_or_IP;

         location /api/ {
             proxy_pass http://localhost:5000/;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
         }

         location / {
             root /path/to/your/react/build;
             try_files $uri /index.html;
         }
     }
     ```

   - Copy the React build files to the directory served by Nginx:
     ```sh
     sudo cp -r job_recommendation_frontend/build/* /var/www/html/
     ```

   - Restart Nginx to apply the changes:
     ```sh
     sudo systemctl restart nginx
     ```

### Step 5: Testing and Finalizing

1. **Test Your Application**:
   - Open a web browser and navigate to your domain or IP.
   - Ensure the job description form is displayed and test the functionality.

2. **Secure Your Application**:
   - Obtain an SSL certificate (you can use Let's Encrypt for free SSL):
     ```sh
     sudo apt install certbot python3-certbot-nginx
     sudo certbot --nginx -d your_domain
     ```

3. **Automate Starting Your Flask App**:
   - Create a systemd service to ensure your Flask app starts on boot:
     ```sh
     sudo nano /etc/systemd/system/flaskapp.service
     ```

   - Add the following content:
     ```ini
     [Unit]
     Description=Gunicorn instance to serve my Flask application
     After=network.target

     [Service]
     User=your_username
     Group=www-data
     WorkingDirectory=/path/to/job_recommendation_backend
     Environment="PATH=/path/to/job_recommendation_backend/venv/bin"
     ExecStart=/path/to/job_recommendation_backend/venv/bin/gunicorn --workers 3 --bind unix:job_recommendation_backend.sock -m 007 wsgi:app

     [Install]
     WantedBy=multi-user.target
     ```

   - Enable and start the service:
     ```sh
     sudo systemctl start flaskapp
     sudo systemctl enable flaskapp
     ```

By following these steps, you will have a fully functional AI-driven job recommendation and resume building website deployed on your GCP VM with Ubuntu OS. Remember to customize and expand the NLP model and recommendations to suit your specific needs and improve the user experience.
