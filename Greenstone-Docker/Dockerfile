#Step1:- Set up frontend dockerfile
FROM nginx 
WORKDIR /frontend 
COPY frontend/ /usr/share/nginx/html

#Step2:- Set up backend dockerfile
FROM python:3.9-slim as backend
WORKDIR /backend
COPY backend / /backend/
RUN pip install --no-cache-dir -r requirements.txt

#Step3:-Combining backend and frontend
FROM nginx:alpine
COPY --from=frontend-build /usr/share/nginx/html /usr/share/nginx/html 
COPY --from=backend /backend /backend
EXPOSE 80 5000                                                       
CMD ["bat", "-c", "python /backend/app.py &nginx -g 'daemon off;'"]
