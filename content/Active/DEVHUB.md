<div class="htb-box-info medium">
  <div class="box-header">
    <img src="https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/8e821c7bbdb90d8520bb597edae70080.png" alt="Machine Name">
    <div class="box-details">
      <h2>Box Info</h2>
      <table>
        <tr><td>OS:</td><td>Linux</td></tr>
        <tr><td>Difficulty:</td><td>Medium</td></tr>
        <tr><td>Release:</td><td>30 May 2026</td></tr>
      </table>
    </div>
  </div>
</div>

The box is active => there is no writeup till it not active...
But I have this short exploatation path:

1.Exposed Python/MCP Developer Tool  
2.Unsafe MCP command/config handling  
3.Backend command execution  
4.Reverse shell as low-privileged service user  
5.Local enumeration of Python, Jupyter, MCP, ports, history, and configs  
6.Discovery of internal tokens, API keys, or localhost-only services  
7.Access to internal MCP/API functionality  
8.Abuse of trusted admin tool or tool-call endpoint  
9.Disclosure of privileged credential  
10.Authentication as root/admin  
11.Full system compromise
