# ESGI M2IW - Projet de Kubernetes

## Présentation du sujet

Le but de ce projet est de migrer une application e-commerce open source 'OroCommerce' d'un docker-compose vers un cluster Kubernetes tout en prenant en compte les bonnes pratiques de déploiement et d'architecture cloud native.

## Présentaion du projet

![architecture](architecture.png)

Namespaces :

- `orocommerce` : Application principale
- `monitoring` : Prometheus, Grafana

Composants :

- OroCommerce (PHP-FPM + Nginx)
- PostgreSQL (StatefulSet)
- WebSocket, Consumer, Cron (pods dédiés)
- Monitoring (Prometheus, Grafana, etc.)
- Ingress HTTP/S

## 🚀 Démarrage Rapide

```bash
# 1. Démarrer minikube
minikube start --driver=docker --memory=4096 --cpus=2
minikube addons enable ingress

# 2. Se placer dans le répertoire du projet
cd helm_chart

# 3. Déployer OroCommerce
helm install orocommerce . --namespace orocommerce --create-namespace

# 4. Déployer le monitoring
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana --namespace monitoring --create-namespace
helm install prometheus prometheus-community/prometheus --namespace monitoring
kubectl apply -f monitoring-ingress.yaml

# 5. Configurer l'accès
echo "127.0.0.1 oro.demo grafana.local prometheus.local" | sudo tee -a /etc/hosts
minikube tunnel  # Laisser ce terminal ouvert !

# 6. Accéder aux services
# OroCommerce: https://oro.demo
# Grafana: https://grafana.local (admin/[voir commande ci-dessous])
# Prometheus: https://prometheus.local
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

## 🏗️ Architecture

### Composants principaux

- **OroCommerce Application** : Application e-commerce PHP-FPM avec Nginx
- **PostgreSQL** : Base de données en StatefulSet
- **Nginx** : Serveur web frontend avec SSL/TLS
- **WebSocket** : Service de communication temps réel
- **Consumer & Cron** : Services de traitement en arrière-plan

### Monitoring Stack

- **Prometheus** : Collecte des métriques
- **Grafana** : Visualisation et dashboards
- **Alertmanager** : Gestion des alertes
- **Node Exporter** : Métriques système
- **Kube State Metrics** : Métriques Kubernetes

## 📋 Prérequis

- **Docker** : Version récente installée
- **Minikube** : v1.30+
- **Helm** : v3.x
- **kubectl** : Compatible avec Kubernetes v1.25+
- **macOS/Linux** : Pour les commandes système

## 🚀 Installation et Démarrage

### 1. Démarrage du cluster Kubernetes

```bash
# Démarrer minikube avec les ressources nécessaires
minikube start --driver=docker --memory=4096 --cpus=2

# Activer l'addon ingress-nginx
minikube addons enable ingress
```

### 2. Déploiement d'OroCommerce

```bash
# Se placer dans le répertoire Helm
cd helm_chart

# Déployer OroCommerce avec Helm
helm install orocommerce . --namespace orocommerce --create-namespace
```

### 3. Configuration du monitoring

```bash
# Ajouter les repositories Helm
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Déployer Grafana
helm install grafana grafana/grafana --namespace monitoring --create-namespace

# Déployer Prometheus
helm install prometheus prometheus-community/prometheus --namespace monitoring

# Appliquer les ingress de monitoring
kubectl apply -f monitoring-ingress.yaml
```

### 4. Configuration de l'accès externe

```bash
# Démarrer le tunnel minikube (IMPORTANT: ne pas mettre en arrière-plan)
# Le tunnel doit rester actif dans un terminal dédié
minikube tunnel

# Dans un NOUVEAU terminal, ajouter les entrées DNS locales
echo "127.0.0.1 oro.demo grafana.local prometheus.local" | sudo tee -a /etc/hosts
```

⚠️ **Important** : Le tunnel minikube doit rester actif et demandera le mot de passe sudo. Ne fermez pas ce terminal !

## 🌐 Accès aux Applications

### URLs d'accès

- **OroCommerce** : https://oro.demo
- **Grafana** : https://grafana.local
- **Prometheus** : https://prometheus.local

### Identifiants Grafana

- **Utilisateur** : `admin`
- **Mot de passe** : Récupérer avec la commande :
  ```bash
  kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
  ```

### ✅ Test de connectivité

```bash
# Tester l'accès aux services (doit retourner des codes 200/302)
curl -k -I https://oro.demo
curl -k -I https://grafana.local
curl -k -I https://prometheus.local
```

## 📊 Vérification du Déploiement

### Vérifier les pods OroCommerce

```bash
kubectl get pods -n orocommerce
```

### Vérifier les pods de monitoring

```bash
kubectl get pods -n monitoring
```

### Vérifier les ingress

```bash
kubectl get ingress -A
```

### Vérifier les services

```bash
kubectl get services -A
```

### Vérifier que tout fonctionne

```bash
# État global du cluster
kubectl get pods -A | grep -E "(Running|Ready)"

# Test des URLs (doit retourner des codes HTTP valides)
curl -k -I https://oro.demo
curl -k -I https://grafana.local
curl -k -I https://prometheus.local

# Vérifier le tunnel
ps aux | grep "minikube tunnel"
```

## Documentation

La migration vers Kubernetes offre des bénéfices clés :

1. **Orchestration**

- **Auto-healing** et **scaling** intégrés
- **Rolling updates** avec rollback

2. **Résilience**

- **Répartition de charge** automatique
- **Meilleure gestion des pannes**

3. **Stateful workloads**

- **StatefulSets** pour bases de données
- **Volumes persistants** natifs

4. **Réseau**

- **Ingress unifié** pour l'exposition des services
- **Load Balancer** intégré (vs port binding manuel)

5. **Monitoring**

- **Métriques système et applicatives** natives
- **Intégration facile** avec Prometheus/Grafana

## Membres du Groupe 6

- CARCIU Mateo
- KAFADAR Cihan
- NGGILI PAUL
- ZAHAF-KRADRA Ilyas
