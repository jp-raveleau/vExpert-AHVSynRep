<html>
<h1>vExpert Session :<br/> DR with AHV Sync Rep and Leap</h1>

<p>The Nutanix AOS 5.17 release offer significant enhancements to Leap for on-premises failover operations, including support for execution of guest scripts and synchronous replication with AHV.</p>
<p><strong>In this lab you will deploy a multi-tier application, protect your VMs, build a Recovery Plan for runbook automation, and perform a failover operation to another Nutanix cluster.</strong></p>
<h2>Staging the Application<h2/>
<h3>Provisioning Your Application</h3>
<ul class="simple">
<li><p>Log in to Prism Central for your <strong>PrimarySite</strong> cluster using the following credentials:</p>
<li><p><strong>Username</strong> - admin</p></li>
<li><p><strong>Password</strong> - nx2Tech123!</p></li>
</ul>
</li>
<li><p>Open <em class="fa fa-bars"></em> <strong>&gt; Administration &gt; Availability Zones</strong> and observe that the cluster has already been paired to another Prism Central instance containing your <strong>SecondarySite</strong> cluster. No action is required to add additional Availability Zones for this lab.</p>
<div class="figure align-default">
<img alt="../../_images/1104.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/1104.png" />
</div>
</li>
<li><p>Open <em class="fa fa-bars"></em> <strong>&gt; Services &gt; Calm</strong> and select <strong>Blueprints</strong> from the sidebar.</p></li>
<li><p>Select the <strong>FiestaApp</strong> Blueprint and click <strong>Actions &gt; Launch</strong>.</p>
<div class="figure align-default">
<img alt="../../_images/249.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/249.png" />
</div>
</li>
<li><p>Fill out the following fields and then click <strong>Create</strong> to begin provisioning your application:</p>
<ul class="simple">
<li><p><strong>Name of the Application</strong> - <em>Initials</em>-FiestaApp</p></li>
<li><p><strong>Initials</strong> - <em>Initials</em></p></li>
<li><p><strong>SQL Password</strong> - <em>What you want</em></p></li>
</ul>
</li>
<li><p>Monitor the status of the application in the <strong>Audit</strong> tab and proceed once your application enters a <strong>Running</strong> state.</p></li>
<li><p>On the <strong>Services</strong> tab, select the <strong>NodeReact</strong> service and note the IP Address. This is the web server hosting the front end of your application.</p></li>
<li><p>Open <a class="reference external" href="http:/">http:/</a>/&lt;<em>NodeReact-VM-IP-Address:5001</em>&gt; in a new browser tab and validate you can access the Fiesta Inventory Management app.</p>
<div class="figure align-default">
<img alt="../../_images/519.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/519.png" />
</div>
</li>
</ol>
</div>
<div class="section" id="installing-nutanix-guest-tools">
<h3>Installing Nutanix Guest Tools<a class="headerlink" href="#installing-nutanix-guest-tools" title="Permalink to this headline">¶</a></h3>
<ol class="arabic">
<li><p>Open <em class="fa fa-bars"></em> <strong>&gt; Virtual Infrastructure &gt; VMs</strong>.</p></li>
<li><p>Select your <em>Initials</em><strong>-WebServer-…</strong> VM and click <strong>Actions &gt; Update</strong>.</p></li>
<li><p>Under <strong>Disks</strong>, click <em class="fa fa-eject"></em> beside <strong>CD-ROM</strong> to unmount the Cloud-Init disk mounted during the Calm deployment.</p></li>
<li><p>Click <strong>Save</strong>.</p></li>
<li><p>Repeat <strong>Steps 2-4</strong> to eject the <strong>CD-ROM</strong> on your <em>Initials</em><strong>-MySQL-…</strong> VM.</p></li>
<li><p>Select both VMs and click <strong>Actions &gt; Install NGT</strong>.</p>
<div class="figure align-default">
<img alt="../../_images/420.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/420.png" />
</div>
</li>
<li><p>Select <strong>Restart as soon as the install is completed</strong> and click <strong>Confirm &amp; Enter Password</strong>.</p>
<div class="figure align-default">
<img alt="../../_images/4b.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/4b.png" />
</div>
</li>
<li><p>Provide the following credentials and click <strong>Done</strong> to begin the NGT installation:</p>
<ul class="simple">
<li><p><strong>User Name</strong> - centos</p></li>
<li><p><strong>Password</strong> - nutanix/4u</p></li>
</ul>
<div class="figure align-default">
<img alt="../../_images/4c.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/4c.png" />
</div>
</li>
<li><p>Once both VMs have rebooted, validate both VMs now have empty CD-ROM drives and <strong>NGT Status</strong> displays <strong>Latest</strong> in Prism Central.</p>
<div class="figure align-default">
<img alt="../../_images/619.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/619.png" />
</div>
</li>
</ol>
</div>
</div>
<div class="section" id="staging-guest-script">
<h2>Staging Guest Script<a class="headerlink" href="#staging-guest-script" title="Permalink to this headline">¶</a></h2>
<p>New in 5.17, Leap allows you to execute scripts within a guest to update configuration files or perform other critical functions as part of the runbook. In this exercise you’ll stage a script on your WebServer VM that will update its configuration file responsible for the MySQL VM connection, allowing the WebServer to connect to the MySQL database after failover to our <strong>SecondarySite</strong> network.</p>
<ol class="arabic">
<li><p>SSH into your <em>Initials</em><strong>-WebServer-…</strong> VM using the following credentials:</p>
<ul class="simple">
<li><p><strong>User Name</strong> - centos</p></li>
<li><p><strong>Password</strong> - nutanix/4u</p></li>
</ul>
</li>
<li><p>Within the VM SSH session, execute the following:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span><span class="nb">cd</span> /usr/local/sbin
sudo wget https://raw.githubusercontent.com/nutanixworkshops/ts2020/master/onpremleap/production_vm_recovery
sudo chmod +x /usr/local/sbin/production_vm_recovery
</pre></div>
</div>
<div class="admonition note">
<p class="admonition-title">Note</p>
<p>Run <code class="docutils literal notranslate"><span class="pre">sudo</span> <span class="pre">cat</span> <span class="pre">/usr/local/sbin/production_vm_recovery</span></code> to view the contents of the failover script.</p>
</div>
</li>
</ol>
</div>
<div class="section" id="creating-a-protection-policy">
<h2>Creating A Protection Policy<a class="headerlink" href="#creating-a-protection-policy" title="Permalink to this headline">¶</a></h2>
<ol class="arabic">
<li><p>In Prism Central, open <em class="fa fa-bars"></em> <strong>&gt; Policies &gt; Protection Policies</strong>.</p></li>
<li><p>Click <strong>Create Protection Policy</strong>.</p></li>
<li><p>Fill out the following fields:</p>
<ul class="simple">
<li><p><strong>Name</strong> - <em>Initials</em>-FiestaProtection</p></li>
<li><p><strong>Primary Cluster(s)</strong> - PrimarySite</p></li>
<li><p><strong>Recovery Location</strong> - PC_10.42.xx.xx</p></li>
<li><p><strong>Target Cluster</strong> - SecondarySite</p></li>
<li><p>Under <strong>Policy Type</strong>, select <strong>Synchronous</strong></p></li>
<li><p>Under <strong>Failure Handling</strong>, select <strong>Automatic</strong></p></li>
<li><p><strong>Timeout After</strong> - 10 Seconds</p></li>
</ul>
<div class="figure align-default">
<img alt="../../_images/719.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/719.png" />
</div>
<div class="admonition note">
<p class="admonition-title">Note</p>
<p>Protection policies can be automatically applied based on Category assignment, allowing VMs to be automatically protected from their initial provisioning. We will not assign categories in this lab.</p>
</div>
</li>
<li><p>Click <strong>Save</strong>.</p></li>
</ol>
</div>
<div class="section" id="assigning-a-protection-policy">
<h2>Assigning A Protection Policy<a class="headerlink" href="#assigning-a-protection-policy" title="Permalink to this headline">¶</a></h2>
<ol class="arabic">
<li><p>In Prism Central, open <em class="fa fa-bars"></em> <strong>&gt; Virtual Infrastructure &gt; VMs</strong>.</p></li>
<li><p>Select both of your VMs and click <strong>Actions &gt; Protect</strong>.</p></li>
<li><p>Select your <em>Initials</em><strong>-FiestaProtection</strong> policy and click <strong>Protect</strong>.</p>
<div class="figure align-default">
<img alt="../../_images/919.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/919.png" />
</div>
</li>
<li><p>In the <strong>VM List</strong>, click <strong>Focus</strong> and select <strong>Data Protection</strong> from the drop down menu.</p>
<div class="figure align-default">
<img alt="../../_images/1018.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/1018.png" />
</div>
</li>
<li><p>Observe the <strong>Protection Status</strong> of each of your VMs move to <strong>Synced</strong>.</p>
<div class="figure align-default">
<img alt="../../_images/1120.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/1120.png" />
</div>
</li>
</ol>
</div>
<div class="section" id="creating-a-recovery-plan">
<h2>Creating A Recovery Plan<a class="headerlink" href="#creating-a-recovery-plan" title="Permalink to this headline">¶</a></h2>
<ol class="arabic">
<li><p>In Prism Central, open <em class="fa fa-bars"></em> <strong>&gt; Policies &gt; Recovery Plans</strong>.</p></li>
<li><p>Click <strong>Create Recovery Plan</strong>.</p></li>
<li><p>Select <strong>PC_10.42.xx.xx</strong> as your <strong>Recovery Location</strong> and click <strong>Proceed</strong>.</p></li>
<li><p>Specify <em>Initials</em><strong>-FiestaRecovery</strong> as your <strong>Recovery Plan Name</strong> and click <strong>Next</strong>.</p></li>
<li><p>Under <strong>Power On Sequence</strong> we will add our VMs in stages to the plan. Click <strong>+ Add Entities</strong>.</p></li>
<li><p>Select your <em>Initials</em><strong>-MySQL-…</strong> VM and click <strong>Add</strong>.</p>
<div class="figure align-default">
<img alt="../../_images/1218.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/1218.png" />
</div>
</li>
<li><p>Click <strong>+ Add New Stage</strong>. Under <strong>Stage 2</strong>, click <strong>+ Add Entities</strong>.</p>
<div class="figure align-default">
<img alt="../../_images/1319.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/1319.png" />
</div>
</li>
<li><p>Select your <em>Initials</em><strong>-WebServer-…</strong> VM and click <strong>Add</strong>.</p></li>
<li><p>Select your <em>Initials</em><strong>-WebServer-…</strong> VM and click <strong>Manage Scripts &gt; Enable</strong>. This will run the <strong>production_vm_recovery</strong> script within the guest VM you staged in a previous exercise.</p>
<div class="figure align-default">
<img alt="../../_images/2116.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/2116.png" />
</div>
<div class="admonition note">
<p class="admonition-title">Note</p>
<p>You can mouse over <strong>Script Path</strong> to see where Leap expects guest scripts for Windows and Linux guests.</p>
</div>
</li>
<li><p>Click <strong>+ Add Delay</strong> between your two stages.</p>
<div class="figure align-default">
<img alt="../../_images/1417.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/1417.png" />
</div>
</li>
<li><p>Specify <strong>60</strong> seconds and click <strong>Add</strong>.</p></li>
<li><p>Click <strong>Next</strong>.</p>
<p>In this step you will map VM networks from your primary site to your recovery site.</p>
</li>
<li><p>Select <strong>VLANx</strong> for <strong>Local AZ Production</strong> and <strong>Local AZ Test Failback</strong> Virtual Networks. Select <strong>VLANx</strong> for <strong>PC_10.42.x.x Production</strong> and <strong>PC_10.42.x.x Test Failback</strong> Virtual Networks.</p>
<div class="figure align-default">
<img alt="../../_images/1515.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/1515.png" />
</div>
</li>
<li><p>Click <strong>Done</strong>.</p></li>
</ol>
</div>
<div class="section" id="performing-an-unplanned-failover">
<h2>Performing An Unplanned Failover<a class="headerlink" href="#performing-an-unplanned-failover" title="Permalink to this headline">¶</a></h2>
<p>Before performing our failover, we’ll make a quick update to our application.</p>
<ol class="arabic">
<li><p>Open <a class="reference external" href="http:/">http:/</a>/&lt;<em>Initials-WebServer-VM-IP-Address:5001</em>&gt; in another browser tab.</p></li>
<li><p>Under <strong>Stores</strong>, click <strong>Add New Store</strong> and fill out the required fields. Validate your new store appears in the UI.</p>
<div class="figure align-default">
<img alt="../../_images/1615.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/1615.png" />
</div>
</li>
<li><p>Log in to Prism Central for your <strong>SecondarySite</strong> (NOT YOUR <strong>PrimarySite</strong> CLUSTER) using the following credentials:</p>
<ul class="simple">
<li><p><strong>Username</strong> - admin</p></li>
<li><p><strong>Password</strong> - nx2Tech123!</p></li>
</ul>
</li>
<li><p>Open <em class="fa fa-bars"></em> <strong>&gt; Policies &gt; Recovery Plans</strong>.</p></li>
<li><p>Select your <em>Initials</em><strong>-FiestaRecovery</strong> plan and click <strong>Actions &gt; Failover</strong>.</p>
<div class="figure align-default">
<img alt="../../_images/1715.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/1715.png" />
</div>
</li>
<li><p>To simulate a true DR event, under <strong>Failover Type</strong>, select <strong>Unplanned Failover</strong> and click <strong>Failover</strong>.</p>
<div class="figure align-default">
<img alt="../../_images/1815.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/1815.png" />
</div>
</li>
<li><p>Ignore any warnings related to Calm categories not found in the Recovery AZ and click <strong>Execute Anyway</strong>.</p></li>
<li><p>Click the <strong>Name</strong> of your Recovery Plan to monitor status of plan execution. Select <strong>Tasks &gt; Failover</strong> for full details.</p>
<div class="figure align-default">
<img alt="../../_images/209.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/d4403954493947f58ec63a8c72be6cd0/i/file/421830c402ed4fa4a1d17d296ff06830/209.png" />
</div>
</li>
<li><p>Once the Recovery Plan reaches 100%, open <em class="fa fa-bars"></em> <strong>&gt; Virtual Infrastructure &gt; VMs</strong> and note the <em>new</em> IP Address of your <em>Initials</em><strong>-WebServer-…</strong>.</p></li>
<li><p>Open <a class="reference external" href="http:/">http:/</a>/&lt;<em>Initials-WebServer-VM-NEW-IP-Address:5001</em>&gt; in another browser tab and verify the change you’d made to your application is present.</p>
<p>Congratulations! You’ve completed your first DR failover with Nutaix AHV, leveraging native Leap runbook capabilities and synchronous replication.</p>
</li>
</ol>
</div>
<div class="section" id="cleanup">
<h2>Cleanup<a class="headerlink" href="#cleanup" title="Permalink to this headline">¶</a></h2>
<p>After validating your lab, please clean up the environment by doing the following:</p>
<ol class="arabic simple">
<li><p>Delete your Recovery Plan and Protection Policy</p></li>
<li><p>Delete your VMs from the <strong>SecondarySite</strong></p></li>
<li><p>Delete your <em>Initials</em><strong>-FiestaApp</strong> application in Calm (<strong>DO NOT DELETE THE BLUEPRINT</strong>) on your <strong>PrimarySite</strong> and validate the VMs have been deleted.</p></li>

</html>