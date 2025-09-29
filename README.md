# ClickHouse Exploration Docker Setup

This repository provides a streamlined way to launch a **ClickHouse** environment using **Docker Compose**. It's ideal for local development, testing, and exploring ClickHouse features without needing to install the database directly on your host machine.

***

## 🚀 Getting Started

### Prerequisites

You need the following installed on your system:

* **[Docker](https://www.docker.com/get-started)**
* **[Docker Compose](https://docs.docker.com/compose/install/)** (often included with modern Docker installations)

### Installation

1.  **Clone the Repository**
    ```bash
    git clone [https://github.com/leonlowzd/clickhouse-exploration.git](https://github.com/leonlowzd/clickhouse-exploration.git)
    cd clickhouse-exploration
    ```

2.  **Start the Stack**

    Run the following command in the root directory where `docker-compose.yml` is located. This will pull the necessary images and start the services (e.g., ClickHouse server).

    ```bash
    cd ./dev
    docker-compose up -d
    ```

***

## 🔗 Accessing the Jupyterlab

We can interface with clickhouse and kafka easily on [Jupyterlab]( http://localhost:8999). Open the notebook, ./notebooks/dem-setup.ipynb to run the demo.