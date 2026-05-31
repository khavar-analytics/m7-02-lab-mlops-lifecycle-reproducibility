```mermaid
graph TD
    
    A[(Data Lake / Feature Store)] -->|dataset_hash: d9f2e3| B[Experimentation & Training Pipeline]
    
    
    B -->|mlflow_run_id: run_8812a| C[Model Evaluation Set]
    C -->|evaluation_report.json| D{Automated Gates Passed?}
    
    
    D -->|Yes: mlflow_stage: Staging| REG_STAGING[Registry: Staging Stage]
    D -->|No: run_status: Rejected| B
    
    REG_STAGING -->|Manual approval ticket_id: 402| REG_PROD[Registry: Production Stage]
    REG_PROD -->|mlflow_stage: Archived| REG_ARCHIVED[Registry: Archived Stage]
    
    
    REG_PROD -->|model_uri: models:/eta/12| K1
    
    subgraph DEPLOYMENT_PHASE [Deployment Pipeline]
        K1[Shadow Deployment] -->|shadow_telemetry.log| K2[Canary Rollout]
    end
    
    
    K2 -->|traffic_weight: 1.0| G[Production Traffic - Triton Server]
    
    
    G -->|inference_payload_stream.json| H[Model Monitoring - Prometheus/Evidently]
    H -->|drift_alert_signal: MAE > 150s| I{Retrain Triggered?}
    
    
    I -->|Yes: trigger_dag_id: eta_retrain| B
    I -->|No: slack_alert_payload| H


    style REG_STAGING fill:#ffeeba,stroke:#ffc107,stroke-width:2px
    style REG_PROD fill:#d4edda,stroke:#28a745,stroke-width:2px
    style REG_ARCHIVED fill:#e2e3e5,stroke:#6c757d,stroke-width:2px
    style DEPLOYMENT_PHASE fill:#f8d7da,stroke:#dc3545,stroke-width:1px