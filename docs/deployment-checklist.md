# Deployment Checklist

A practical checklist for verifying the basic readiness of an application before deployment.

## 1. Application Status

- [ ] Verify that the application service or container is running.
- [ ] Check the process status and confirm that the application starts without errors.
- [ ] Confirm that all required dependencies are available.

## 2. Network and Ports

- [ ] Verify that the application is listening on the expected port.
- [ ] Check that required network connections are available.
- [ ] Confirm that firewall rules allow the necessary traffic.

## 3. HTTP Health Checks

- [ ] Send an HTTP request to the application.
- [ ] Verify that the application returns the expected HTTP status code.
- [ ] Check the application's health endpoint, if available.

## 4. Logs

- [ ] Review service or container logs for startup errors.
- [ ] Investigate recent warnings and error messages.

## 5. Disk and Resources

- [ ] Verify that sufficient disk space is available.
- [ ] Check CPU and memory usage for unexpected resource consumption.

## 6. Configuration and Security

- [ ] Verify that required environment variables are configured.
- [ ] Confirm that configuration files and permissions are correct.
- [ ] Ensure that passwords, tokens, and secrets are not stored in the Git repository.

## 7. Final Verification

- [ ] Confirm that the application can be accessed through its intended endpoint.
- [ ] Verify that the deployed version matches the intended release.
- [ ] Document any issues and confirm that a rollback plan is available.
