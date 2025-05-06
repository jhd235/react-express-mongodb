# Deploying to Google Cloud Platform

This guide will help you deploy the application to Google Cloud Platform using Cloud Run and Cloud Build.

## Prerequisites

1. Install the [Google Cloud SDK](https://cloud.google.com/sdk/docs/install)
2. Create a Google Cloud Project
3. Enable required APIs:
   - Cloud Run API
   - Cloud Build API
   - Container Registry API
   - Cloud SQL Admin API (if using Cloud SQL)

## Setup Steps

1. **Initialize Google Cloud SDK**
   ```bash
   gcloud init
   ```

2. **Set your project ID**
   ```bash
   export PROJECT_ID=your-project-id
   ```

3. **Enable required APIs**
   ```bash
   gcloud services enable run.googleapis.com
   gcloud services enable cloudbuild.googleapis.com
   gcloud services enable containerregistry.googleapis.com
   ```

4. **Create a MongoDB Atlas Cluster**
   - Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
   - Create a new cluster
   - Get your connection string
   - Add your IP to the whitelist

5. **Set up environment variables**
   ```bash
   # For backend
   gcloud run services update backend \
     --update-env-vars MONGODB_URI=your_mongodb_connection_string \
     --region us-central1

   # For frontend
   gcloud run services update frontend \
     --update-env-vars REACT_APP_API_URL=https://backend-xxxxx-uc.a.run.app \
     --region us-central1
   ```

6. **Deploy using Cloud Build**
   ```bash
   gcloud builds submit --config cloudbuild.yaml
   ```

## Post-Deployment

1. **Get the deployed URLs**
   ```bash
   gcloud run services list
   ```

2. **Update CORS settings**
   - Go to your backend service in Cloud Run
   - Update the CORS settings to allow requests from your frontend URL

3. **Set up custom domain (optional)**
   ```bash
   gcloud beta run domain-mappings create \
     --service frontend \
     --domain your-domain.com \
     --region us-central1
   ```

## Monitoring

1. **View logs**
   ```bash
   gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=frontend"
   gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=backend"
   ```

2. **Monitor metrics**
   - Go to Cloud Run in the Google Cloud Console
   - Select your service
   - Click on "Monitoring" tab

## Troubleshooting

1. **Check service status**
   ```bash
   gcloud run services describe frontend --region us-central1
   gcloud run services describe backend --region us-central1
   ```

2. **View build logs**
   ```bash
   gcloud builds log $(gcloud builds list --limit=1 --format='value(id)')
   ```

## Security Considerations

1. **Set up IAM roles**
   - Use least privilege principle
   - Assign appropriate roles to service accounts

2. **Enable security features**
   - Enable Cloud Audit Logs
   - Set up VPC Service Controls if needed
   - Configure Cloud Armor for DDoS protection

## Cost Optimization

1. **Set up budget alerts**
   ```bash
   gcloud billing budgets create \
     --billing-account=your-billing-account \
     --budget-amount=100USD \
     --display-name="Monthly Budget"
   ```

2. **Configure auto-scaling**
   - Set minimum instances to 0 for cost savings
   - Configure maximum instances based on your needs
   - Set up appropriate concurrency limits 