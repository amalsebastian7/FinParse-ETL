# 🚀 Deployment, Operations & Runbook

## 1. Build and Run Instructions

### 1.1 Local Development
1. Clone the repository: `git clone https://github.com/amalsebastian7/StatementProcessorJava.git`
2. Configure environment: Copy `.env.example` to `.env` and add your Google Gemini API key.
3. Build the project: `mvn clean install`
4. Run the application: 
   ```bash
   java -jar target/finparse-etl-1.0.0-jar-with-dependencies.jar --dir=~/Documents/BankStatements
   ```

## 2. Production Automation (Linux / NAS / Raspberry Pi)
For users running this as an automated daily job.

### 2.1 Systemd Service Definition
Create a file at `/etc/systemd/system/finparse.service`:

```ini
[Unit]
Description=FinParse ETL Financial Pipeline
After=network.target

[Service]
Type=oneshot
User=pi
EnvironmentFile=/opt/finparse/.env
ExecStart=/usr/bin/java -jar /opt/finparse/finparse.jar --dir=/mnt/nas/statements
StandardOutput=append:/var/log/finparse.log
StandardError=append:/var/log/finparse.error.log
```

### 2.2 Systemd Timer (Cron Replacement)
Create a file at `/etc/systemd/system/finparse.timer` to run it daily at 2:00 AM:

```ini
[Unit]
Description=Run FinParse ETL Daily

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

## 3. Incident Runbook

| Alert / Error Log | Root Cause | Resolution Steps |
| :--- | :--- | :--- |
| `java.sql.SQLException: database is locked` | SQLite file is being read by another process (e.g., DB Browser or an exporter script). | Terminate the reading process. SQLite requires exclusive write access during the ETL phase. |
| `HTTP 429 Too Many Requests` | Gemini Free Tier quota exceeded (15 RPM). | The application's resilience4j circuit breaker should handle this. If it fails, increase the `api.batch.sleep_ms` in `application.properties`. |
| `OpenCSVException: Unparseable date format` | The bank has changed their CSV export format. | Update the specific bank's parser adapter class to accommodate the new `DateTimeFormatter` pattern. |
