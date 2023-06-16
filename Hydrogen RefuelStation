using Sandbox.ModAPI.Ingame;
using System.Collections.Generic;
using VRage.Game.ModAPI.Ingame;
using VRageMath;

namespace IngameScript
{
    public class Program : MyGridProgram
    {
        const double MinimumIceAmount = 1000; // Minimum ice amount to trigger refilling (in liters)
        const double MaximumHydrogenLevel = 0.95; // Maximum hydrogen level to stop generator (0.0 - 1.0)

        List<IMyTerminalBlock> generators = new List<IMyTerminalBlock>();
        List<IMyTerminalBlock> cargoContainers = new List<IMyTerminalBlock>();
        List<IMyTerminalBlock> hydrogenTanks = new List<IMyTerminalBlock>();
        int delayTicks = 60; // Delay between each iteration (in ticks)
        int currentTick = 0;
        IMyTextPanel lcdPanel;
        bool isShipConnected = false;

        public Program()
        {
            Runtime.UpdateFrequency = UpdateFrequency.Update1;
            lcdPanel = GridTerminalSystem.GetBlockWithName("LCD Panel") as IMyTextPanel; // Replace "LCD Panel" with the name of your LCD panel block
        }

        public void Main(string argument, UpdateType updateType)
        {
            // Initialization
            if (string.IsNullOrEmpty(argument))
            {
                Echo("[+] Init");
                Echo("[+] Detecting Blocks");
                GetBlocksOfType<IMyGasGenerator>(generators);
                GetBlocksOfType<IMyCargoContainer>(cargoContainers);
                GetBlocksOfType<IMyGasTank>(hydrogenTanks);
                Echo("[+] Blocks Detected");
                GetIceAmount();

            }

            // Check if all required blocks exist
            if (generators.Count == 0 || cargoContainers.Count == 0 || hydrogenTanks.Count == 0)
            {
                Echo("Error: Missing required blocks.");
                return;
            }

            // Only update on timer or argument
            if (updateType == UpdateType.Update1 || updateType == UpdateType.Trigger)
            {
                currentTick++;

                if (currentTick >= delayTicks)
                {
                    currentTick = 0;

                    // Check if cargo containers have enough ice
                    double iceAmount = GetIceAmount();
                    if (iceAmount < MinimumIceAmount)
                    {
                        // Not enough ice, start filling
                        if (isShipConnected)
                        {
                            // A ship is connected, stop filling the generators
                            TurnOffGenerators();
                        }
                        else
                        {
                            // No ship connected, turn off generators
                            TurnOffGenerators();
                            if (iceAmount <= 0)
                            {
                                // No ice available, display a warning
                                lcdPanel.WriteText("WARNING: No ice available!");
                            }
                        }
                        return;
                    }
                    
                    // Enough ice, check hydrogen level
                    double hydrogenLevel = GetHydrogenLevel();
                    if (hydrogenLevel < MaximumHydrogenLevel)
                    {
                        // Hydrogen level is below maximum, turn on generators
                        TurnOnGenerators();
                    }
                    else
                    {
                        // Hydrogen level is at maximum, turn off generators
                        TurnOffGenerators();
                    }
                }
            }
        }

        // Get the current ice amount in all cargo containers
        double GetIceAmount()
        {
            double iceAmount = 0;

            foreach (var container in cargoContainers)
            {
                if (container.HasInventory)
                {
                    var inventory = container.GetInventory();
                    var items = new List<MyInventoryItem>();

                    inventory.GetItems(items);

                    foreach (var item in items)
                    {
                        if (item.Type.TypeId.ToString().Contains("MyObjectBuilder_Ore") && item.Type.SubtypeId.Contains("Ice"))
                        {
                            iceAmount += (double)item.Amount;
                        }
                    }
                }
            }

            return iceAmount;
        }

        // Get the current hydrogen level in the tanks
        double GetHydrogenLevel()
        {
            double totalCapacity = 0;
            double totalFilled = 0;

            foreach (var tank in hydrogenTanks)
            {
                if (tank is IMyGasTank)
                {
                    IMyGasTank gasTank = (IMyGasTank)tank;
                    totalCapacity += gasTank.Capacity;
                    totalFilled += gasTank.FilledRatio * gasTank.Capacity;
                }
            }

            if (totalCapacity > 0)
                return totalFilled / totalCapacity;
            else
                return 0;
        }

        // Fill the generators with ice from cargo containers
        void FillGeneratorWithIce()
        {
            foreach (var generator in generators)
            {
                if (generator.HasInventory)
                {
                    var generatorInventory = generator.GetInventory();
                    var items = new List<MyInventoryItem>();

                    foreach (var container in cargoContainers)
                    {
                        if (container.HasInventory)
                        {
                            var containerInventory = container.GetInventory();
                            containerInventory.GetItems(items);

                            foreach (var item in items)
                            {
                                if (item.Type.TypeId.ToString().Contains("MyObjectBuilder_Ore") && item.Type.SubtypeId.Contains("Ice"))
                                {
                                    containerInventory.TransferItemTo(generatorInventory, item);
                                }
                            }
                        }
                    }
                }
            }
        }

        // Turn on the generators
        void TurnOnGenerators()
        {
            foreach (var generator in generators)
            {
                generator.ApplyAction("OnOff_On");
            }
        }

        // Turn off the generators
        void TurnOffGenerators()
        {
            foreach (var generator in generators)
            {
                generator.ApplyAction("OnOff_Off");
            }
        }

        // Get blocks of a specific type and store them in a list
        void GetBlocksOfType<T>(List<IMyTerminalBlock> blocks) where T : class
        {
            blocks.Clear();
            GridTerminalSystem.GetBlocksOfType(blocks, b => b is T);
        }
    }
}
