# apnacollage-demo
This is my first Git Repository.
<br>
Author-Rifat Islam 
package java.lang;

import android.system.ErrnoException;
import android.system.StructPasswd;
import android.system.StructUtsname;
import dalvik.system.VMRuntime;
import dalvik.system.VMStack;
import java.io.BufferedInputStream;
import java.io.Console;
import java.io.FileDescriptor;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.io.IOException;
import java.io.PrintStream;
import java.nio.channels.Channel;
import java.nio.channels.spi.SelectorProvider;
import java.util.AbstractMap;
import java.util.Collections;
import java.util.HashMap;
import java.util.Locale;
import java.util.Map;
import java.util.Properties;
import java.util.Set;
import libcore.icu.ICU;
import libcore.io.Libcore;

/**
 * Provides access to system-related information and resources including
 * standard input and output. Enables clients to dynamically load native
 * libraries. All methods of this class are accessed in a static way and the
 * class itself can not be instantiated.
 *
 * @see Runtime
 */
public final class System {

    /**
     * Default input stream.
     */
    public static final InputStream in;

    /**
     * Default output stream.
     */
    public static final PrintStream out;

    /**
     * Default error output stream.
     */
    public static final PrintStream err;

    private static final String PATH_SEPARATOR = ":";
    private static final String LINE_SEPARATOR = "\n";
    private static final String FILE_SEPARATOR = "/";

    private static final Properties unchangeableSystemProperties;
    private static Properties systemProperties;

    /**
     * Dedicated lock for GC / Finalization logic.
     */
    private static final Object lock = new Object();

    /**
     * Whether or not we need to do a GC before running the finalizers.
     */
    private static boolean runGC;

    /**
     * If we just ran finalization, we might want to do a GC to free the finalized objects.
     * This lets us do gc/runFinlization/gc sequences but prevents back to back System.gc().
     */
    private static boolean justRanFinalization;

    static {
        err = new PrintStream(new FileOutputStream(FileDescriptor.err));
        out = new PrintStream(new FileOutputStream(FileDescriptor.out));
        in = new BufferedInputStream(new FileInputStream(FileDescriptor.in));
        unchangeableSystemProperties = initUnchangeableSystemProperties();
        systemProperties = createSystemProperties();

        addLegacyLocaleSystemProperties();
    }

    private static void addLegacyLocaleSystemProperties() {
        final String locale = getProperty("user.locale", "");
        if (!locale.isEmpty()) {
            Locale l = Locale.forLanguageTag(locale);
            setUnchangeableSystemProperty("user.language", l.getLanguage());
            setUnchangeableSystemProperty("user.region", l.getCountry());
            setUnchangeableSystemProperty("user.variant", l.getVariant());
        } else {
            // If "user.locale" isn't set we fall back to our old defaults of
            // language="en" and region="US" (if unset) and don't attempt to set it.
            // The Locale class will fall back to using user.language and
            // user.region if unset.
            final String language = getProperty("user.language", "");
            final String region = getProperty("user.region", "");

            if (language.isEmpty()) {
                setUnchangeableSystemProperty("user.language", "en");
            }

            if (region.isEmpty()) {
                setUnchangeableSystemProperty("user.region", "US");
            }
        }
    }

    /**
     * Sets the standard input stream to the given user defined input stream.
     *
     * @param newIn
     *            the user defined input stream to set as the standard input
     *            stream.
     */
    public static void setIn(InputStream newIn) {
        setFieldImpl("in", "Ljava/io/InputStream;", newIn);
    }

    /**
     * Sets the standard output stream to the given user defined output stream.
     *
     * @param newOut
     *            the user defined output stream to set as the standard output
     *            stream.
     */
    public static void setOut(PrintStream newOut) {
        setFieldImpl("out", "Ljava/io/PrintStream;", newOut);
