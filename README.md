Предлагаемый в проекте Vagrantfile служит для автоматического создания рейд-массива и добавления конфигурации в файл fstab.
1. Зануляем суперблоки

mdadm --zero-superblock --force /dev/sd{b,c,d,e,f,g}

2. Создаем рейд-массив (рейд пять из шести дисков)

mdadm --create --verbose /dev/md0 -l 5 -n 6 /dev/sd{b,c,d,e,f,g}

3. Проверяем состояние массива

cat /proc/mdstat

4. Создаем файл mdadm.conf

echo "DEVICE partitions" > /etc/mdadm/mdadm.conf

mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf

5. Создаем GPT раздел на собранном массиве

parted -s /dev/md0 mklabel gpt

6. Создаем 5 партиций

parted /dev/md0 mkpart primary ext4 0% 20%

parted /dev/md0 mkpart primary ext4 20% 40%

parted /dev/md0 mkpart primary ext4 40% 60%

parted /dev/md0 mkpart primary ext4 60% 80%

parted /dev/md0 mkpart primary ext4 80% 100%

7. Создаем файловую систему на них

for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done

8. Монтируем по каталогам

mkdir -p /raid/part{1,2,3,4,5}

for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done

9. Вносим изменения в файл /etc/fstab

echo "#MY ARRAY DEVICES" >> /etc/fstab

for i in $(seq 1 5);do  blkid /dev/md0p$i | awk '{sub(/"/,"",$2); sub(/"/,"",$2); print $2, "/raid/part"'$i', "ext4", "defaults", "0", "0"}' >> /etc/fstab; done
          

Используйте этот [Vagrantfile](Vagrantfile) - для тестового стенда.
