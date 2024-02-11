# Integrate - Spark

FLAML has integrated Spark for distributed training. There are two main aspects of integration with Spark:
- Use Spark ML estimators for AutoML.
- Use Spark to run training in parallel spark jobs.

## Spark ML Estimators

FLAML integrates estimators based on Spark ML models. These models are trained in parallel using Spark, so we called them Spark estimators. To use these models, you first need to organize your data in the required format.