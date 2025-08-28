Exactement ✅
👉 Dans **VirtualBox**, rajouter un **deuxième disque** est souvent **plus simple et plus sûr** que de redimensionner celui existant (qui peut casser la table GPT, comme tu viens de voir).

---

## 🔹 Avantages d’ajouter un deuxième disque

* Pas de risque de casser la table GPT ou le boot.
* Pas besoin de manipuler GParted ou `gdisk`.
* Tu gardes ton disque système intact.
* Tu peux utiliser le second disque uniquement pour les **données** (ex. `/data`, `/home`, `D:` sous Windows).

---

## 🔹 Étapes (VirtualBox GUI)

### 1. Éteindre la VM

Toujours obligatoire avant de changer le matériel virtuel.

---

### 2. Ajouter un nouveau disque

1. Ouvre VirtualBox → **Paramètres** de ta VM.
2. Onglet **Stockage**.
3. Clique sur le **contrôleur SATA** → bouton **Ajouter un disque dur**.
4. Choisis **Créer un nouveau disque** (VDI).
5. Choisis la taille (par ex. 50 Go) et le mode dynamique.
6. Valide.

---

### 3. Démarrer la VM

* L’OS invité va détecter un **nouveau disque vierge**.
* Tu devras **le formater et le monter**.

---

## 🔹 Initialiser le disque dans l’OS invité

### 🔸 Sous Linux (Ubuntu/Debian)

1. Vérifie que le disque est présent :

   ```bash
   lsblk
   ```

   → Tu verras sûrement `/dev/sdb`.

2. Crée une partition et un système de fichiers :

   ```bash
   sudo parted /dev/sdb mklabel gpt
   sudo parted -a opt /dev/sdb mkpart primary ext4 0% 100%
   sudo mkfs.ext4 /dev/sdb1
   ```

3. Monte le disque (exemple `/data`) :

   ```bash
   sudo mkdir /data
   sudo mount /dev/sdb1 /data
   ```

4. (Option) Ajoute au `/etc/fstab` pour un montage automatique :

   ```bash
   echo "/dev/sdb1 /data ext4 defaults 0 2" | sudo tee -a /etc/fstab
   ```

---

### 🔸 Sous Windows

1. Ouvre **Gestion des disques** (`diskmgmt.msc`).
2. Windows détecte le disque non initialisé → choisis **GPT**.
3. Crée un **nouveau volume simple**.
4. Formate en **NTFS** et assigne une lettre (ex. `D:`).

---

## 🔹 Conclusion

👉 Si ton but est juste **avoir plus de place**, **ajouter un nouveau disque** est **plus rapide, moins risqué, et plus propre**.
👉 Si ton but est **agrandir la partition système (C:/ ou /)**, il faut par contre passer par un redimensionnement.

---

Veux-tu que je t’écrive un **pas-à-pas pratique** pour ajouter un deuxième disque de données **dans tes VMs Ubuntu (vm1, vm2, vm3)** avec toutes les commandes prêtes ?
