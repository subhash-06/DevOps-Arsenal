# Production-ready Dockerfile for static website
FROM nginx:alpine

# Metadata
LABEL maintainer="raavis2002@gmail.com"
LABEL description="CI/CD Demo Application"
LABEL version="1.0.0"

# Remove default nginx content
RUN rm -rf /usr/share/nginx/html/*

# Copy application files
COPY index.html /usr/share/nginx/html/

# Copy custom nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Fix permissions for non-root nginx user
RUN chown -R nginx:nginx /usr/share/nginx/html && \
    chown -R nginx:nginx /var/cache/nginx && \
    chmod -R 755 /usr/share/nginx/html && \
    touch /var/run/nginx.pid && \
    chown -R nginx:nginx /var/run/nginx.pid && \
    chown -R nginx:nginx /var/log/nginx

# Switch to non-root user
USER nginx

# Expose HTTP port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost/health || exit 1

# Start nginx in foreground
CMD ["nginx", "-g", "daemon off;"]