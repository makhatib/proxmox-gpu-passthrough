# proxmox-gpu-passthrough

🧠 الدليل النهائي لـ PCI/GPU Passthrough في Proxmox (موثوق 100%)

youtube 
https://youtu.be/mCGsKsde84Q

في هذا الفيديو بشرح لكم خطوة بخطوة كيف تمرر كرت شاشة (أو أي كرت PCI) إلى Virtual Machine داخل Proxmox بطريقة مضمونة ومستقرة، مع دعم كامل لـ Windows أو Linux VMs.

🧩 الخطوة 1: تفعيل IOMMU من BIOS
✅ فعل الخيارات التالية من إعدادات BIOS/UEFI:

Intel:

✅ Intel VT-x

✅ Intel VT-d

AMD:

✅ SVM Mode

✅ IOMMU Controller

💡 تأكد أن الإقلاع على وضع UEFI وليس Legacy/CSM.

⚙️ الخطوة 2: تعديل GRUB


nano /etc/default/grub

✔️ بدّل السطر:

GRUB_CMDLINE_LINUX_DEFAULT="quiet"

🔸 إذا جهازك Intel:


GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt pcie_acs_override=downstream,multifunction video=vesafb:off,efifb:off nofb nomodeset"

🔸 إذا AMD:


GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt pcie_acs_override=downstream,multifunction video=vesafb:off,efifb:off nofb nomodeset"


🔄 ثم:


update-grub

🧬 الخطوة 3: تفعيل VFIO Modules

nano /etc/modules

✔️ أضف السطور التالية:


vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
🔐 الخطوة 4: إعدادات استقرار وتوافق



echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf


🚫 الخطوة 5: منع Proxmox من استخدام كرت الشاشة


echo "blacklist nouveau" > /etc/modprobe.d/blacklist-nvidia.conf
echo "blacklist radeon" >> /etc/modprobe.d/blacklist-nvidia.conf
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist-nvidia.conf

🧠 الخطوة 6: جلب PCI IDs وربطها بـ VFIO


عرض الكروت:


lspci -nn

استخراج الـ IDs:




lspci -n -s 01:00

تعديل ملف VFIO:



nano /etc/modprobe.d/vfio.conf
✔️ أضف السطر التالي (مثال):



options vfio-pci ids=10de:2232,10de:228b disable_vga=1


🔄 الخطوة 7: تحديث الـ initramfs وإعادة التشغيل



update-initramfs -u -k all
reboot


🧪 الخطوة 8: تأكيد الربط


lspci -nnk -d 10de:2232

✔️ النتيجة المطلوبة:


Kernel driver in use: vfio-pci


🖥️ الخطوة 9: إضافة الكرت إلى VM
🔸 من واجهة Proxmox GUI:

افتح VM

ادخل على Hardware → Add → PCI Device

اختر الكرت الصحيح

فعّل:

✅ All Functions

✅ PCI-Express

✅ Primary GPU

✅ (اختياري) ROM-Bar

غيّر Display إلى virtio-gpu (مهم)

