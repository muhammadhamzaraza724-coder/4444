package com.armormaking;

import net.fabricmc.api.ClientModInitializer;
import net.fabricmc.fabric.api.client.event.lifecycle.v1.ClientTickEvents;
import net.minecraft.client.KeyMapping;
import net.minecraft.client.Minecraft;
import net.minecraft.world.inventory.ClickType;
import net.minecraft.world.inventory.MerchantMenu;
import net.minecraft.world.item.ItemStack;
import net.minecraft.world.item.enchantment.Enchantments;
import net.minecraft.world.item.trading.MerchantOffer;
import net.minecraft.world.item.trading.MerchantOffers;
import org.lwjgl.glfw.GLFW;

import java.util.List;

/**
 * Entry point. K toggles. While ON: talks to the nearest villager / wandering trader,
 * buys enchanted books (Unbreaking III, Mending, Protection IV) if you have the payment,
 * then closes the trade screen and repeats.
 */
public class ArmorMakingMod implements ClientModInitializer {
    private static KeyMapping toggleKey;
    private static boolean enabled = false;
    private static int cooldown = 0;

    @Override
    public void onInitializeClient() {
        toggleKey = Compat.registerKey("key.armormaking.toggle", GLFW.GLFW_KEY_K);
        ClientTickEvents.END_CLIENT_TICK.register(this::tick);
    }

    private void tick(Minecraft mc) {
        if (mc.player == null || mc.level == null || mc.gameMode == null) return;

        while (toggleKey.consumeClick()) {
            enabled = !enabled;
            cooldown = 0;
            Compat.chat(enabled ? "Started" : "Stopped");
        }
        if (!enabled) return;

        if (cooldown > 0) {
            cooldown--;
            return;
        }

        // Trade screen is open -> do the buying
        if (mc.player.containerMenu instanceof MerchantMenu menu) {
            buyWanted(menu);
            Compat.closeMenuAsync();
            cooldown = 20;
            return;
        }

        // No screen open -> find a merchant and open the trade screen
        if (mc.screen == null) {
            List<Integer> ids = Compat.merchantIds();
            if (ids.isEmpty()) {
                Compat.chat("No villager within reach, stopping.");
                enabled = false;
                return;
            }
            Compat.useEntity(ids.get(0));
            cooldown = 15;
        }
    }

    private static boolean wanted(ItemStack result) {
        return Compat.bookLevel(result, Enchantments.UNBREAKING) >= 3
                || Compat.bookLevel(result, Enchantments.MENDING) >= 1
                || Compat.bookLevel(result, Enchantments.PROTECTION) >= 4;
    }

    private static void buyWanted(MerchantMenu menu) {
        MerchantOffers offers = menu.getOffers();
        for (int i = 0; i < offers.size(); i++) {
            MerchantOffer offer = offers.get(i);
            if (offer.isOutOfStock()) continue;
            ItemStack result = offer.getResult();
            if (!wanted(result)) continue;
            if (!Compat.hasStack(offer.getCostA()) || !Compat.hasStack(offer.getCostB())) continue;

            Compat.selectTrade(menu, i);
            // slot 2 = trade output; shift-click moves it to the inventory
            Compat.click(menu, 2, 0, ClickType.QUICK_MOVE);
            Compat.chat("Bought: " + Compat.name(result.getItem()));
        }
    }
}
