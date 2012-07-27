// Decompiled by DJ v3.12.12.96 Copyright 2011 Atanas Neshkov  Date: 7/27/2012 12:53:39 AM
// Home Page: http://members.fortunecity.com/neshkov/dj.html  http://www.neshkov.com/dj.html - Check often for new version!
// Decompiler options: packimports(3) 
// Source File Name:   Commandgive.java

package com.earth2me.essentials.commands;

import com.earth2me.essentials.*;
import com.earth2me.essentials.craftbukkit.InventoryWorkaround;
import java.util.Locale;
import org.bukkit.*;
import org.bukkit.command.CommandSender;
import org.bukkit.enchantments.Enchantment;
import org.bukkit.entity.Player;
import org.bukkit.inventory.ItemStack;

// Referenced classes of package com.earth2me.essentials.commands:
//            EssentialsCommand, NotEnoughArgumentsException, Commandenchant

public class Commandgive extends EssentialsCommand
{

    public Commandgive()
    {
        super("give");
    }

    public void run(Server server, CommandSender sender, String commandLabel, String args[])
        throws Exception
    {
        if(args.length < 2)
            throw new NotEnoughArgumentsException();
        ItemStack stack = ess.getItemDb().get(args[1]);
        String itemname = stack.getType().toString().toLowerCase(Locale.ENGLISH).replace("_", "");
        if((sender instanceof Player) && (ess.getSettings().permissionBasedItemSpawn() ? !ess.getUser(sender).isAuthorized("essentials.give.item-all") && !ess.getUser(sender).isAuthorized((new StringBuilder()).append("essentials.give.item-").append(itemname).toString()) && !ess.getUser(sender).isAuthorized((new StringBuilder()).append("essentials.give.item-").append(stack.getTypeId()).toString()) : !ess.getUser(sender).isAuthorized("essentials.itemspawn.exempt") && !ess.getUser(sender).canSpawnItem(stack.getTypeId()).booleanValue()))
            throw new Exception(I18n._("cantSpawnItem", new Object[] {
                itemname
            }));
        User giveTo = getPlayer(server, args, 0);
        if(args.length > 3 && Util.isInt(args[2]) && Util.isInt(args[3]))
        {
            stack.setAmount(Integer.parseInt(args[2]));
            stack.setDurability(Short.parseShort(args[3]));
        } else
        if(args.length > 2 && Integer.parseInt(args[2]) > 0)
            stack.setAmount(Integer.parseInt(args[2]));
        else
        if(ess.getSettings().getDefaultStackSize() > 0)
            stack.setAmount(ess.getSettings().getDefaultStackSize());
        else
        if(ess.getSettings().getOversizedStackSize() > 0 && giveTo.isAuthorized("essentials.oversizedstacks"))
            stack.setAmount(ess.getSettings().getOversizedStackSize());
        if(args.length > 3)
        {
            for(int i = Util.isInt(args[3]) ? 4 : 3; i < args.length; i++)
            {
                String split[] = args[i].split("[:+',;.]", 2);
                if(split.length < 1)
                    continue;
                Enchantment enchantment = Commandenchant.getEnchantment(split[0], (sender instanceof Player) ? ess.getUser(sender) : null);
                int level;
                if(split.length > 1)
                    level = Integer.parseInt(split[1]);
                else
                    level = enchantment.getMaxLevel();
                stack.addEnchantment(enchantment, level);
            }

        }
        if(stack.getType() == Material.AIR)
            throw new Exception(I18n._("cantSpawnItem", new Object[] {
                "Air"
            }));
        String itemName = stack.getType().toString().toLowerCase(Locale.ENGLISH).replace('_', ' ');
        sender.sendMessage((new StringBuilder()).append(ChatColor.BLUE).append("Giving ").append(stack.getAmount()).append(" of ").append(itemName).append(" to ").append(giveTo.getDisplayName()).append(".").toString());
        if(giveTo.isAuthorized("essentials.oversizedstacks"))
            InventoryWorkaround.addItem(giveTo.getInventory(), true, ess.getSettings().getOversizedStackSize(), new ItemStack[] {
                stack
            });
        else
            InventoryWorkaround.addItem(giveTo.getInventory(), true, new ItemStack[] {
                stack
            });
        giveTo.updateInventory();
    }
}