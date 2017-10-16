# EasySQL
Manage SQL Tables in H2 with less code

```java
import com.jagrosh.easysql.*;
import com.jagrosh.easysql.columns.*;
import java.time.Instant;

public class LastSeenManager extends DataManager {
    
    public final static SQLColumn<Long> USER_ID = new LongColumn("USER_ID", false, 0, true);
    public final static SQLColumn<Instant> LAST_ACTIVE = new InstantColumn("LAST_ACTIVE", false, Instant.ofEpochSecond(0L));
    
    public LastSeenManager(DatabaseConnector connector) {
        super(connector, "LAST_SEEN");
    }
    
    public synchronized void setActive(long userId) {
        readWrite(selectAll(USER_ID.is(userId)), results -> {
            if(results.next()) {
                LAST_ACTIVE.updateValue(results, Instant.now());
                results.updateRow();
            } else {
                results.moveToInsertRow();
                USER_ID.updateValue(results, userId);
                LAST_ACTIVE.updateValue(results, Instant.now());
                results.insertRow();
            }
        });
    }
    
    public synchronized Instant lastActive(User user) {
        return read(select(USER_ID.is(user.getId()), LAST_ACTIVE), results -> {
            if(results.next())
                return LAST_ACTIVE.getValue(results);
            return null;
        });
    }
}
```
