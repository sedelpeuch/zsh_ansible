## Testing the Playbook in a Docker Environment

To test the playbook in a Docker environment, follow these steps:

1. **Use an Existing Docker Image**: Use an existing Docker image with Ansible pre-installed.

    ```sh
    docker pull williamyeh/ansible:ubuntu18.04
    ```

2. **Run the Docker Container**: Run the Docker container and mount your playbook directory.

    ```sh
    docker run --rm -it -v $(pwd):/ansible/playbooks williamyeh/ansible:ubuntu18.04 ansible-playbook /ansible/playbooks/main.yml -c local
    ```

3. **Run the Docker Container and Keep it Running**: To inspect the results after the playbook execution, keep the container running.

    ```sh
    docker run --rm -it -v $(pwd):/ansible/playbooks williamyeh/ansible:ubuntu18.04 /bin/bash -c "ansible-playbook /ansible/playbooks/main.yml -c local && exec /bin/bash"
    ```

This will use an existing Docker image with Ansible installed, run the playbook inside the container without requiring SSH, and keep the container running to allow you to inspect the results.
