# mysql_sys.wait_classes_global_by_latency

SELECT 
    SUBSTRING_INDEX(`performance_schema`.`events_waits_summary_global_by_event_name`.`EVENT_NAME`,
            '/',
            3) AS `event_class`,
    SUM(`performance_schema`.`events_waits_summary_global_by_event_name`.`COUNT_STAR`) AS `total`,
    FORMAT_PICO_TIME(SUM(`performance_schema`.`events_waits_summary_global_by_event_name`.`SUM_TIMER_WAIT`)) AS `total_latency`,
    FORMAT_PICO_TIME(MIN(`performance_schema`.`events_waits_summary_global_by_event_name`.`MIN_TIMER_WAIT`)) AS `min_latency`,
    FORMAT_PICO_TIME(IFNULL((SUM(`performance_schema`.`events_waits_summary_global_by_event_name`.`SUM_TIMER_WAIT`) / NULLIF(SUM(`performance_schema`.`events_waits_summary_global_by_event_name`.`COUNT_STAR`),
                            0)),
                    0)) AS `avg_latency`,
    FORMAT_PICO_TIME(MAX(`performance_schema`.`events_waits_summary_global_by_event_name`.`MAX_TIMER_WAIT`)) AS `max_latency`
FROM
    `performance_schema`.`events_waits_summary_global_by_event_name`
WHERE
    ((`performance_schema`.`events_waits_summary_global_by_event_name`.`SUM_TIMER_WAIT` > 0)
        AND (`performance_schema`.`events_waits_summary_global_by_event_name`.`EVENT_NAME` <> 'idle'))
GROUP BY SUBSTRING_INDEX(`performance_schema`.`events_waits_summary_global_by_event_name`.`EVENT_NAME`,
        '/',
        3)
ORDER BY SUM(`performance_schema`.`events_waits_summary_global_by_event_name`.`SUM_TIMER_WAIT`) DESC
