# playbook_poller

Playbook écrit pour mes pollers centreon sous CentOS

Ce que fait ce playbook :

<ul>
<li>Vérification de la version du moteur centengine :</li>
<pre>
      name: version du moteur centengine
      shell: centengine -V | awk '{print $5}' | head -n 1
      register: release
</pre>
<li>Mise à jour des paquets :</li>
<pre>
- name: Mise à jour des paquet
  yum: name=* state=latest
</pre>
<li>Vérification de la version de Centengine après mise à jour :</li>
<pre>
- name: Vérification de la version de Centengine après mise à jour
  shell:  centengine -V | awk '{print $5}' | head -n 1
  register: new_release
</pre>
<li>Affichage de la version de Centengine :</li>
<pre>
- name: Affichage de la version de Centengine
  debug: msg="Version de Centengine {{ new_release.stdout_lines }}"
</pre>
<li>Notification de la mise à niveau de la version de Centengine :</li>
<pre>
- name: Notification de la mise à niveau de la version de Centengine
  debug: msg="PVE à changé de version {{ release.stdout }} à {{ new_release.stdout }}"
  when: release.stdout != new_release.stdout
</pre>
<li>Vérification des services à redémarrer :</li>
<pre>
- name: vérification des services à redémarrer
  shell: needs-restarting | awk '{print $3}'
  register: services
</pre>
<li>Liste des service à redémarrer :</li>
<pre>
- name: Liste des service à redémarrer
  debug: msg="{{ services.stdout_lines | count }} services à redémarrer ({{ services.stdout_lines | join (', ') }})"
</pre>
<li>Redémarage du poller, si des services doivent être redémarrés :</li>
<pre>
- name: Redémarage des services
  shell: reboot
  when: "{{ services.stdout_lines | count }} != 0"
  </pre>
</ul>

Pour la dernière action, je n'ai pas trouver comment redémarrer proprement les services, 
sans paser par l'étape du reboot du poller.
Si quelqu'un connait la solution ou uen piste, je sui preneur, merci.
