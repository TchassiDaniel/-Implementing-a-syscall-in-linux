<div align="center"> <h1>TP4 - ASR2 - ENS Lyon - L3IF - 2020-2021</h1>
  <h2>Implémentation d’un syscall<br>
Alain Tchana (alain.tchana@ens-lyon.fr)<br>
(Note: ce sujet a été rédigé par Stella Bitchebe)</h2>
</div>

Objectif général

L’objectif du Tp est d’implanter le syscall make_vcpu() qui:

Prend en paramètre le PID d'un processus
Marque tous les threads de ce processus comme étant des vCPUs.
Écrit dans le log du noyau linux les différentes opérations qu’il effectue.
Ensuite il faudra réaliser un programme en user-space qui crée des threads et
appelle ce syscall.

Indications relatives à l’implémentation du syscall
Vous devez utiliser la machine virtuelle (VM) créée lors du TP-1 pour recompiler
votre noyau linux (en suivant bien sûr les instructions de ce même TP-1) après
l’ajout du syscall.

De façon générale, pour implémenter un nouveau syscall dans le noyau linux il faut
créer un nouveau dossier pour votre syscall y ajouter votre code (les .c et .h), faire
un Makefile pour le compiler et ajouter votre syscall au Makefile global du kernel.
Cependant, il est préférable, lorsque cela est possible, de réutiliser au maximum le
code ou les fichiers déjà existant dans le noyau linux! Dans votre cas, vous allez
rajouter l’implémentation de votre syscall dans un fichier existant (i.e. on ne va pas
créer de nouveaux fichiers .h et .c, ni modifier de Makefile). Vous écrirez par
exemple l’implémentation (le code) de votre syscall dans le fichier
kernel/sched/core.c en utilisant la macro SYSCALL_DEFINE1 qui permet de définir un
syscall. Ici le chiffre 1 représente le nombre de paramètres du syscall. Recherchez
une utilisation de cette directive dans ce fichier pour vous en inspirer.
Sachez aussi que les threads et les processus sont manipulés par le kernel sous la
forme d’une structure de données: task_struct, définie dans le fichier /include/linux/
sched.h. Pour marquer un thread (task) comme vCPU, vous ajouterez un champ dans
cette structure (qui indiquera par un 1 ou 0 si le processus est un vCPU ou pas). Vous
pourrez le nommer par exemple “is_vcpu”.
Vous aurez besoin de quelques APIs ou fonctions prédéfinies fournies par le noyau
pour pouvoir manipuler les processus:


Rechercher un processus à partir de son pid: find_process_by_pid
Lire et modifier la structure d’un processus de manière sécurisée:
rcu_read_lock, rcu_read_unlock
Parcourir la liste des threads d’un processus: for_each_threadVous aurez aussi besoin des champs suivants de la structure task_struct:

pid: qui est le pid du processus ou du thread (bref d’un task)
tgid: tous les threads d’un processus ont le même tgid qui est égal au pid du
task père. Et pour le processus père, pid = tgid.
La déclaration d’un nouveau syscall se fait dans les fichiers suivants:
-arch/x86/entry/syscall/syscall_64.tbl: qui est la table des syscalls. Le numéro
défini dans ce fichier est “le numéro du syscall”, celui qui sera utilisé pour
l’invoquer dans une appli c.
![image](https://github.com/TchassiDaniel/-Implementing-a-syscall-in-linux/assets/149546306/709eba3c-2fe1-4827-8bfb-4062f6535179)
-include/linux/syscall.h: contient la déclaration des fonctions qui implantent
les syscalls
![image](https://github.com/TchassiDaniel/-Implementing-a-syscall-in-linux/assets/149546306/2a0c98f9-031b-4e4d-a478-7b6c2eb44e74)

La définition de notre syscall est fait dans core.c:
![image](https://github.com/TchassiDaniel/-Implementing-a-syscall-in-linux/assets/149546306/3824661c-5a4e-42a8-b427-18595f9972a3)

Dans l’application créée pour tester votre syscall, utilisez la fonction syscall(int
number, ...) - vous pouvez regarder le man.
![image](https://github.com/TchassiDaniel/-Implementing-a-syscall-in-linux/assets/149546306/a00265b3-3b07-4138-b9db-76eb13ab6429)

Ici 333 est le numéro que nous avons défini pour notre syscall dans le fichier
syscall_64.tbl
![image](https://github.com/TchassiDaniel/-Implementing-a-syscall-in-linux/assets/149546306/5cee9cb2-7acb-498a-9680-9de9de45686b)

Pour vérifier l’exécution de votre syscall, dans l’implémentation vous pourrez faire
des printk (équivalent kernel de printf) dans votre syscall. Et ces prints seront
visibles dans les logs du kernel : sudo dmesg (ajouter un TAG afin de filtrer, grep,
facilement les logs).
