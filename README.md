# Domain_Join
Programitically join a machine to a Windows active directory

using System;
using System.Configuration;
using System.Diagnostics;
using System.Management;
using System.Windows.Forms;

namespace AD_Join
{
    internal static class Program
    {
        private static void Main(string[] args)
        {
            // Defined Variables$
            int JOIN_DOMAIN = 1;
            int ACCT_CREATE = 2;
            // int ACCT_DELETE = 4; For future implementation

            // Defined Strings
            // Username must have rights for AD Join
            // Set in App.Config
            string domain = ConfigurationManager.AppSettings["Domain"];
            string password = ConfigurationManager.AppSettings["Password"];
            string username = ConfigurationManager.AppSettings["User"];
            string destinationOU = ConfigurationManager.AppSettings["DestinationOU"];

            //Parameters
            int parameters = JOIN_DOMAIN | ACCT_CREATE;

            // The args being passed in a specific order
            object[] methodArgs = { domain, password, username, destinationOU, parameters };

            // ManagementObject and set Options
            ManagementObject computerSystem = new ManagementObject("Win32_ComputerSystem.Name=\"" + Environment.MachineName + "\"");
            computerSystem.Scope.Options.Authentication = AuthenticationLevel.PacketPrivacy;
            computerSystem.Scope.Options.Impersonation = ImpersonationLevel.Impersonate;
            computerSystem.Scope.Options.EnablePrivileges = true;
            try
            {
                //Method JoinDomainOrWorkgroup passing the parameters
                object Oresult = computerSystem.InvokeMethod("JoinDomainOrWorkgroup", methodArgs);

                // Returned as int, Casting for coversion
                int result = Convert.ToInt32(Oresult);

                // If result is 0 then the computer is joined ::High Five::
                if (result == 0)
                {
                    MessageBox.Show("Joined Succesfully!");
                    return;
                }
                else
                {
                    // Error List
                    string strErrorDescription = " ";
                    switch (result)
                    {
                        case 5:
                            strErrorDescription = "Process needs escalated permissions";
                            break;

                        case 87:
                            strErrorDescription = "The parameter is incorrect";
                            break;

                        case 110:
                            strErrorDescription = "The system cannot open the specified object";
                            break;

                        case 1326:
                            strErrorDescription = "Logon failure: unknown username or bad password";
                            break;

                        case 1355:
                            strErrorDescription = "The specified domain either does not exist or could not be contacted";
                            break;

                        case 2691:
                            strErrorDescription = "This machine is already joined to the domain";
                            break;

                        case 2692:
                            strErrorDescription = "This machine is not currently joined to a domain";
                            break;

                        default:
                            strErrorDescription = "We're gonna be real with you. We have no idea what happened here!";
                            break;
                    }

                    MessageBox.Show(result + "\n" + strErrorDescription);
                    Console.WriteLine("Press any Key to continue");
                    Console.ReadLine();
                    return;
                }
            }
            catch (Exception ex)
            { MessageBox.Show(ex.Message + "\n" + ex.StackTrace); }

            var psi = new ProcessStartInfo("reboot", "/r /t 0");
            psi.CreateNoWindow = true;
            psi.UseShellExecute = false;
            Process.Start(psi);
        }
    }
